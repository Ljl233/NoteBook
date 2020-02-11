# 前言
ViewPager 现在Android开发中使用广泛，基本每一款App都是ViewPager+Fragment使用，所以我想通过源码来看看具体是怎么实现的。

具体内容包括，ViewPager的布局测量，与Fragment的交互，对Fragment生命周期的影响。

# ViewPager的绘制
view的绘制流程是测量(measure)->布局(layout)->绘制(draw)

我们就从这几个方法来理解Viewpager是如何绘制的

## onMeasure()

- 这个方法主要就是对自己本身的测量，和调用child的measure()方法来进行child的测量，向child提供widthSpec, heightSpec参数，对gravity进行了处理

```java
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        // For simple implementation, our internal size is always 0.
        // We depend on the container to specify the layout size of
        // our view.  We can't really know what it is since we will be
        // adding and removing different arbitrary views and do not
        // want the layout to change as this happens.
        setMeasuredDimension(getDefaultSize(0, widthMeasureSpec),
                getDefaultSize(0, heightMeasureSpec));

        final int measuredWidth = getMeasuredWidth();
        final int maxGutterSize = measuredWidth / 10;
        //这是个啥？ mGutterSize
        mGutterSize = Math.min(maxGutterSize, mDefaultGutterSize);

        // Children are just made to fill our space.
        int childWidthSize = measuredWidth - getPaddingLeft() - getPaddingRight();
        int childHeightSize = getMeasuredHeight() - getPaddingTop() - getPaddingBottom();

        /*
         * Make sure all children have been properly measured. Decor views first.
         * Right now we cheat and make this less complicated by assuming decor
         * views won't intersect. We will pin to edges based on gravity.
         */
        int size = getChildCount();
        for (int i = 0; i < size; ++i) {
            final View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                if (lp != null && lp.isDecor) {
                    final int hgrav = lp.gravity & Gravity.HORIZONTAL_GRAVITY_MASK;
                    final int vgrav = lp.gravity & Gravity.VERTICAL_GRAVITY_MASK;
                    int widthMode = MeasureSpec.AT_MOST;
                    int heightMode = MeasureSpec.AT_MOST;

                    //是否在水平或者垂直上有设置gravity
                    boolean consumeVertical = vgrav == Gravity.TOP || vgrav == Gravity.BOTTOM;
                    boolean consumeHorizontal = hgrav == Gravity.LEFT || hgrav == Gravity.RIGHT;

                    if (consumeVertical) {
                        widthMode = MeasureSpec.EXACTLY;
                    } else if (consumeHorizontal) {
                        heightMode = MeasureSpec.EXACTLY;
                    }

                    int widthSize = childWidthSize;
                    int heightSize = childHeightSize;
                    if (lp.width != LayoutParams.WRAP_CONTENT) {
                        widthMode = MeasureSpec.EXACTLY;
                        if (lp.width != LayoutParams.MATCH_PARENT) {
                            widthSize = lp.width;
                        }
                    }
                    if (lp.height != LayoutParams.WRAP_CONTENT) {
                        heightMode = MeasureSpec.EXACTLY;
                        if (lp.height != LayoutParams.MATCH_PARENT) {
                            heightSize = lp.height;
                        }
                    }
                    final int widthSpec = MeasureSpec.makeMeasureSpec(widthSize, widthMode);
                    final int heightSpec = MeasureSpec.makeMeasureSpec(heightSize, heightMode);
                    child.measure(widthSpec, heightSpec);

                    if (consumeVertical) {
                        childHeightSize -= child.getMeasuredHeight();
                    } else if (consumeHorizontal) {
                        childWidthSize -= child.getMeasuredWidth();
                    }
                }
            }
        }

        mChildWidthMeasureSpec = MeasureSpec.makeMeasureSpec(childWidthSize, MeasureSpec.EXACTLY);
        mChildHeightMeasureSpec = MeasureSpec.makeMeasureSpec(childHeightSize, MeasureSpec.EXACTLY);

        // Make sure we have created all fragments that we need to have shown.
        mInLayout = true;
        populate();
        mInLayout = false;

        // Page views next.
        size = getChildCount();//使用fragment的时候两次size不一样？？？？？？？？？？？？？？？？？？？
        for (int i = 0; i < size; ++i) {
            final View child = getChildAt(i);
            if (child.getVisibility() != GONE) {
                if (DEBUG) {
                    Log.v(TAG, "Measuring #" + i + " " + child + ": " + mChildWidthMeasureSpec);
                }

                final LayoutParams lp = (LayoutParams) child.getLayoutParams();
                if (lp == null || !lp.isDecor) {
                    final int widthSpec = MeasureSpec.makeMeasureSpec(
                            (int) (childWidthSize * lp.widthFactor), MeasureSpec.EXACTLY);
                    child.measure(widthSpec, mChildHeightMeasureSpec);
                }
            }
        }
    }
```
## populate()

```java

void populate(int newCurrentItem) {
    ItemInfo oldCurInfo = null;
    if (mCurItem != newCurrentItem) {
        oldCurInfo = infoForPosition(mCurItem);
        mCurItem = newCurrentItem;
}

    if (mAdapter == null) {
        sortChildDrawingOrder();
        return;
    }

    // Bail now if we are waiting to populate.  This is to hold off
    // on creating views from the time the user releases their finger to
    // fling to a new position until we have finished the scroll to
    // that position, avoiding glitches from happening at that point.
    if (mPopulatePending) {
        if (DEBUG) Log.i(TAG, "populate is pending, skipping for now...");
        sortChildDrawingOrder();
        return;
    }

    // Also, don't populate until we are attached to a window.  This is to
    // avoid trying to populate before we have restored our view hierarchy
    // state and conflicting with what is restored.
    if (getWindowToken() == null) {
        return;
    }

    mAdapter.startUpdate(this);

    final int pageLimit = mOffscreenPageLimit;
    final int startPos = Math.max(0, mCurItem - pageLimit);
    final int N = mAdapter.getCount();
    final int endPos = Math.min(N - 1, mCurItem + pageLimit);

    if (N != mExpectedAdapterCount) {
        String resName;
        try {
            resName = getResources().getResourceName(getId());
        } catch (Resources.NotFoundException e) {
            resName = Integer.toHexString(getId());
        }
        throw new IllegalStateException("The application's PagerAdapter changed the adapter's"
                + " contents without calling PagerAdapter#notifyDataSetChanged!"
                + " Expected adapter item count: " + mExpectedAdapterCount + ", found: " + N
                + " Pager id: " + resName
                + " Pager class: " + getClass()
                + " Problematic adapter: " + mAdapter.getClass());
    }

    // Locate the currently focused item or add it if needed.
    int curIndex = -1;
    ItemInfo curItem = null;
    for (curIndex = 0; curIndex < mItems.size(); curIndex++) {
        final ItemInfo ii = mItems.get(curIndex);
        if (ii.position >= mCurItem) {
            if (ii.position == mCurItem) curItem = ii;
            break;
        }
    }

    if (curItem == null && N > 0) {
        curItem = addNewItem(mCurItem, curIndex);
    }

    // Fill 3x the available width or up to the number of offscreen
    // pages requested to either side, whichever is larger.
    // If we have no current item we have no work to do.
    if (curItem != null) {
        float extraWidthLeft = 0.f;
        int itemIndex = curIndex - 1;
        ItemInfo ii = itemIndex >= 0 ? mItems.get(itemIndex) : null;
        final int clientWidth = getClientWidth();
        final float leftWidthNeeded = clientWidth <= 0 ? 0 :
                2.f - curItem.widthFactor + (float) getPaddingLeft() / (float) clientWidth;
        for (int pos = mCurItem - 1; pos >= 0; pos--) {
            if (extraWidthLeft >= leftWidthNeeded && pos < startPos) {
                if (ii == null) {
                    break;
                }
                if (pos == ii.position && !ii.scrolling) {
                    mItems.remove(itemIndex);
                    mAdapter.destroyItem(this, pos, ii.object);
                    if (DEBUG) {
                        Log.i(TAG, "populate() - destroyItem() with pos: " + pos
                                + " view: " + ((View) ii.object));
                    }
                    itemIndex--;
                    curIndex--;
                    ii = itemIndex >= 0 ? mItems.get(itemIndex) : null;
                }
            } else if (ii != null && pos == ii.position) {
                extraWidthLeft += ii.widthFactor;
                itemIndex--;
                ii = itemIndex >= 0 ? mItems.get(itemIndex) : null;
            } else {
                ii = addNewItem(pos, itemIndex + 1);
                extraWidthLeft += ii.widthFactor;
                curIndex++;
                ii = itemIndex >= 0 ? mItems.get(itemIndex) : null;
            }
        }

        float extraWidthRight = curItem.widthFactor;
        itemIndex = curIndex + 1;
        if (extraWidthRight < 2.f) {
            ii = itemIndex < mItems.size() ? mItems.get(itemIndex) : null;
            final float rightWidthNeeded = clientWidth <= 0 ? 0 :
                    (float) getPaddingRight() / (float) clientWidth + 2.f;
            for (int pos = mCurItem + 1; pos < N; pos++) {
                if (extraWidthRight >= rightWidthNeeded && pos > endPos) {
                    if (ii == null) {
                        break;
                    }
                    if (pos == ii.position && !ii.scrolling) {
                        mItems.remove(itemIndex);
                        mAdapter.destroyItem(this, pos, ii.object);
                        if (DEBUG) {
                            Log.i(TAG, "populate() - destroyItem() with pos: " + pos
                                    + " view: " + ((View) ii.object));
                        }
                        ii = itemIndex < mItems.size() ? mItems.get(itemIndex) : null;
                    }
                } else if (ii != null && pos == ii.position) {
                    extraWidthRight += ii.widthFactor;
                    itemIndex++;
                    ii = itemIndex < mItems.size() ? mItems.get(itemIndex) : null;
                } else {
                    ii = addNewItem(pos, itemIndex);
                    itemIndex++;
                    extraWidthRight += ii.widthFactor;
                    ii = itemIndex < mItems.size() ? mItems.get(itemIndex) : null;
                }
            }
        }

        calculatePageOffsets(curItem, curIndex, oldCurInfo);

        mAdapter.setPrimaryItem(this, mCurItem, curItem.object);
    }

    if (DEBUG) {
        Log.i(TAG, "Current page list:");
        for (int i = 0; i < mItems.size(); i++) {
            Log.i(TAG, "#" + i + ": page " + mItems.get(i).position);
        }
    }

    mAdapter.finishUpdate(this);

    // Check width measurement of current pages and drawing sort order.
    // Update LayoutParams as needed.
    final int childCount = getChildCount();
    for (int i = 0; i < childCount; i++) {
        final View child = getChildAt(i);
        final LayoutParams lp = (LayoutParams) child.getLayoutParams();
        lp.childIndex = i;
        if (!lp.isDecor && lp.widthFactor == 0.f) {
            // 0 means requery the adapter for this, it doesn't have a valid width.
            final ItemInfo ii = infoForChild(child);
            if (ii != null) {
                lp.widthFactor = ii.widthFactor;
                lp.position = ii.position;
            }
        }
    }
    sortChildDrawingOrder();

    if (hasFocus()) {
        View currentFocused = findFocus();
        ItemInfo ii = currentFocused != null ? infoForAnyChild(currentFocused) : null;
        if (ii == null || ii.position != mCurItem) {
            for (int i = 0; i < getChildCount(); i++) {
                View child = getChildAt(i);
                ii = infoForChild(child);
                if (ii != null && ii.position == mCurItem) {
                    if (child.requestFocus(View.FOCUS_FORWARD)) {
                        break;
                    }
                }
            }
        }
    }
}

```