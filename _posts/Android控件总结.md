---
title: Android控件总结
date: 2018-08-20 22:25:54
tags:
categories: Android_基础
---

在wms服务中，我们可以直接使用它的api来创建一个窗口，显示出来并且通过InputEventReceiver接收输入事件。但是这种方法太原始，
1. 需要完全的Android源码环境
2. 需要自己去处理UI元素的测量，布局和绘制
3. 还需要处理InputEventReceiver事件，分发到合适的窗口
4. wms来的各种回调
<!--more-->
因此Android提供了控件系统来帮我们完成各种各样的控件的创建。更高级一点的创建方式是获取 WindowManager,然后通过addView()方法得到一个可以交互的有界面的窗口

关于WindowManager的一个类图
![WindowManager结构](Android控件总结/WindowManager结构.jpg)

## 窗口添加view的过程
然后是为窗口添加view所发生的调用过程

![WindowManager结构](Android控件总结/WindowManager.addView过程.jpg)

我们跟进WindManager.addView()过程，可以看到最终通过RootViewImpl.addView()调用到了PerformTraversals()。除此之外，
requestLayout()也会导致 PerformTraversals() 被调用：
```Java
    @Override
    public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {   //在layout过程中会被设置为true
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }

     void scheduleTraversals() {
        if (!mTraversalScheduled) {   //屏蔽重复的调用
            mTraversalScheduled = true;
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            //即使performTraversals()执行比较快，在一次垂直同步的时间里最多只会调用performTraversals()一次
            mChoreographer.postCallback(Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);  
            if (!mUnbufferedInputDispatch) {
                scheduleConsumeBatchedInput();
            }
            notifyRendererOfFramePending();
            pokeDrawLockIfNeeded();
        }
    }

    final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
            doTraversal();
        }
    }

    void doTraversal() {
        if (mTraversalScheduled) {
            mTraversalScheduled = false;
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);

            if (mProfile) {
                Debug.startMethodTracing("ViewAncestor");
            }

            performTraversals();

            if (mProfile) {
                Debug.stopMethodTracing();
                mProfile = false;
            }
        }
    }
```

`performTraversals()`主要经过了以下几个阶段：

1. 预测量阶段，这里会对控件树第一次进行测量，计算出为了显示控件树所希望的窗口大小，将会依次调用view和子类的onMeasure()方法。
2. 布局窗口阶段，这里会把上一步得到的数据发给wms，wms会对窗口进行重新布局，计算出系统实际上给客户端的窗口大小并返回。
3. 最终测量阶段，这里只能接受wms给的窗口大小，并根据这个大小最终计算出控件树的实际大小，将会依次调用view和子类的onMeasure()方法。
4. 布局控件树， 上一步得到测量结果这里就可以计算出控件的位置，将会依次调用View和子类的onLayout()方法
5. 绘制。将会依次调用View和子类的onDraw()方法

![WindowManager结构](Android控件总结/performTraversals.jpg)

需要注意的是， onMeasure和onLayout,onDraw都是可以跳过的

### 测量阶段(1,2,3)

第一次测量的时候，使用的 desiredWindowWidth 和 desiredWindowHeight 就是屏幕的大小，后面再测量的时候使用的就是上次测量得出的窗口大小了。在measureHierarchy可以看到，如果performMeasure()的结果带有标记MEASURED_STATE_TOO_SMALL，performMeasure()有可能被调用多次。

```Java
performTraversals(){
...
    if (mFirst) {
        mFullRedrawNeeded = true;
        mLayoutRequested = true;

        final Configuration config = mContext.getResources().getConfiguration();
        if (shouldUseDisplaySize(lp)) {
            // NOTE -- system code, won't try to do compat mode.
            ...
        } else {
            desiredWindowWidth = dipToPx(config.screenWidthDp);
            desiredWindowHeight = dipToPx(config.screenHeightDp);
        }
    else{
        desiredWindowWidth = frame.width();
        desiredWindowHeight = frame.height();
        if (desiredWindowWidth != mWidth || desiredWindowHeight != mHeight) {
            if (DEBUG_ORIENTATION) Log.v(mTag, "View " + host + " resized to: " + frame);
            mFullRedrawNeeded = true;
            mLayoutRequested = true;
            windowSizeMayChange = true;
        }
    }

    //没有开始绘制的时候可能会有消息过来，这时候要存在一个队列里面，开始绘制了再执行
    getRunQueue().executeActions(mAttachInfo.mHandler);

    boolean layoutRequested = mLayoutRequested && (!mStopped || mReportNextDraw);
    if (layoutRequested) {

        final Resources res = mView.getContext().getResources();

        if (mFirst) {
            // make sure touch mode code executes by setting cached value
            // to opposite of the added touch mode.
            mAttachInfo.mInTouchMode = !mAddedTouchMode;
            ensureTouchModeLocally(mAddedTouchMode);
        } else {
            ...
        }

        // Ask host how big it wants to be
        windowSizeMayChange |= measureHierarchy(host, lp, res,
                desiredWindowWidth, desiredWindowHeight);
    }


    private boolean measureHierarchy(final View host, final WindowManager.LayoutParams lp,
            final Resources res, final int desiredWindowWidth, final int desiredWindowHeight) {
        int childWidthMeasureSpec;
        int childHeightMeasureSpec;
        boolean windowSizeMayChange = false;

        if (DEBUG_ORIENTATION || DEBUG_LAYOUT) Log.v(mTag,
                "Measuring " + host + " in display " + desiredWindowWidth
                + "x" + desiredWindowHeight + "...");

        boolean goodMeasure = false;
        if (lp.width == ViewGroup.LayoutParams.WRAP_CONTENT) {
            // On large screens, we don't want to allow dialogs to just
            // stretch to fill the entire width of the screen to display
            // one line of text.  First try doing the layout at a smaller
            // size to see if it will fit.
            final DisplayMetrics packageMetrics = res.getDisplayMetrics();
            res.getValue(com.android.internal.R.dimen.config_prefDialogWidth, mTmpValue, true);
            int baseSize = 0;
            if (mTmpValue.type == TypedValue.TYPE_DIMENSION) {
                baseSize = (int)mTmpValue.getDimension(packageMetrics);
            }
            if (DEBUG_DIALOG) Log.v(mTag, "Window " + mView + ": baseSize=" + baseSize
                    + ", desiredWindowWidth=" + desiredWindowWidth);
            if (baseSize != 0 && desiredWindowWidth > baseSize) {
                childWidthMeasureSpec = getRootMeasureSpec(baseSize, lp.width);
                childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
                performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
                if (DEBUG_DIALOG) Log.v(mTag, "Window " + mView + ": measured ("
                        + host.getMeasuredWidth() + "," + host.getMeasuredHeight()
                        + ") from width spec: " + MeasureSpec.toString(childWidthMeasureSpec)
                        + " and height spec: " + MeasureSpec.toString(childHeightMeasureSpec));
                if ((host.getMeasuredWidthAndState()&View.MEASURED_STATE_TOO_SMALL) == 0) {
                    goodMeasure = true;
                } else {
                    // Didn't fit in that size... try expanding a bit.
                    baseSize = (baseSize+desiredWindowWidth)/2;
                    if (DEBUG_DIALOG) Log.v(mTag, "Window " + mView + ": next baseSize="
                            + baseSize);
                    childWidthMeasureSpec = getRootMeasureSpec(baseSize, lp.width);
                    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
                    if (DEBUG_DIALOG) Log.v(mTag, "Window " + mView + ": measured ("
                            + host.getMeasuredWidth() + "," + host.getMeasuredHeight() + ")");
                    if ((host.getMeasuredWidthAndState()&View.MEASURED_STATE_TOO_SMALL) == 0) {
                        if (DEBUG_DIALOG) Log.v(mTag, "Good!");
                        goodMeasure = true;
                    }
                }
            }
        }

        if (!goodMeasure) {
            childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
            childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
            performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
            if (mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight()) {
                windowSizeMayChange = true;
            }
        }

        if (DBG) {
            System.out.println("======================================");
            System.out.println("performTraversals -- after measure");
            host.debug();
        }

        return windowSizeMayChange;
    }

    private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        ...
        mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
        ...
    }

    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        ...
        if (forceLayout || needsLayout) {
            // first clears the measured dimension flag
            mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

            resolveRtlPropertiesIfNeeded();

            int cacheIndex = forceLayout ? -1 : mMeasureCache.indexOfKey(key);
            if (cacheIndex < 0 || sIgnoreMeasureCache) {
                // measure ourselves, this should set the measured dimension flag back
                onMeasure(widthMeasureSpec, heightMeasureSpec);
                mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            } else {
                long value = mMeasureCache.valueAt(cacheIndex);
                // Casting a long to int drops the high 32 bits, no mask needed
                setMeasuredDimensionRaw((int) (value >> 32), (int) value);
                mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
            }
            ...
            mPrivateFlags |= PFLAG_LAYOUT_REQUIRED; //measure里面会对flag进行设置，所以不能直接重写measure()，而是写onMeasure()
        }
        ...
    }
}
```


### 布局控件树
这里主要的工作内容是 performLayout() 和  mWindowSession.setTransparentRegion().

整个窗口默认都是透明区域，当普通控件加入时，控件会把自己的区域从透明区域移除掉，而SurfaceView会把自己的区域添加到当前窗口的透明区域中。
随后这个区域会被设置给wms,在surfaceFlinger对surface进行混合的时候，窗口的透明区域将会被忽略掉

需要注意的是在layout方法里面，会调用到setFrame来检查布局坐标是否变化，如果发生变化，就会调用invalidate(),此时一定会调用onDraw()

```Java
performTraversals(){
    ... //测量阶段(1,2,3)
    final boolean didLayout = layoutRequested && (!mStopped || mReportNextDraw);
    boolean triggerGlobalLayoutListener = didLayout|| mAttachInfo.mRecomputeGlobalAttributes;
    if (didLayout) {
        performLayout(lp, mWidth, mHeight);
        ...
        if (!mTransparentRegion.equals(mPreviousTransparentRegion)) {
            mPreviousTransparentRegion.set(mTransparentRegion);
            mFullRedrawNeeded = true;
            // reconfigure window manager
            try {
                mWindowSession.setTransparentRegion(mWindow, mTransparentRegion);
            } catch (RemoteException e) {
            }
        }
    }
    ... //绘制阶段
}

private void performLayout(WindowManager.LayoutParams lp, int desiredWindowWidth,int desiredWindowHeight) {
        ...
        final View host = mView;
        ...
        host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
        ...
}

public void layout(int l, int t, int r, int b) {
    if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
        onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
        mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
    }
    ...
    //注意这里，setOpticalFrame和setFrame都会调用到setFrame，里面可能会调用invalidate(),此时一定会调用onDraw()
    boolean changed = isLayoutModeOptical(mParent) ?setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);
    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
        onLayout(changed, l, t, r, b);

        if (shouldDrawRoundScrollbar()) {
            if(mRoundScrollbarRenderer == null) {
                mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
            }
        } else {
            mRoundScrollbarRenderer = null;
        }

        mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;  ////layout里面会对flag进行设置，所以不能直接重写layout()，而是写onLayout()
        ...
    }
    ...
}

protected boolean setFrame(int left, int top, int right, int bottom) {
    boolean changed = false;
    if (mLeft != left || mRight != right || mTop != top || mBottom != bottom) {
        changed = true;
        ...
        boolean sizeChanged = (newWidth != oldWidth) || (newHeight != oldHeight);
        // Invalidate our old position
        invalidate(sizeChanged);
        ...
        if (sizeChanged) {
            sizeChange(newWidth, newHeight, oldWidth, oldHeight);
        }
        ...
    return changed;
}
```

### 绘制控件
1. 如果view不可见，不需要绘制
2. 如果surface是新创建的(比如从不可见到可见，此时会新创建surface)，不需要绘制，调用scheduleTraversals()下次再绘制。
3. 绘制的时候仅仅会对需要重绘的区域进行绘制，这部分区域称为脏区域。如果mDirty为空，有可能不会进行绘制。在控件中表示为`PFLAG_DIRTY`和`PFLAG_DIRTY_OPAQUE`,表示这个控件中是否有需要重绘的区域。其中`PFLAG_DIRTY_OPAQUE`表示此区域是不透明的，如果是整个控件，那就是意思是可以不用控件的底层背景，提高绘制效率。

```Java
performTraversals(){
    ... //测量阶段(1,2,3)
    ... //布局阶段
    boolean cancelDraw = mAttachInfo.mTreeObserver.dispatchOnPreDraw() || !isViewVisible;
    if (!cancelDraw && !newSurface) {
        if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
            for (int i = 0; i < mPendingTransitions.size(); ++i) {
                mPendingTransitions.get(i).startChangingAnimations();
            }
            mPendingTransitions.clear();
        }
        performDraw();
    } else {
        if (isViewVisible) {
            scheduleTraversals();
        }
       ...
    }
}

private void performDraw() {
        if (mAttachInfo.mDisplayState == Display.STATE_OFF && !mReportNextDraw) {
            return;
        } else if (mView == null) {
            return;
        }

        final boolean fullRedrawNeeded = mFullRedrawNeeded;
        mFullRedrawNeeded = false;

        mIsDrawing = true;
        Trace.traceBegin(Trace.TRACE_TAG_VIEW, "draw");
        try {
            draw(fullRedrawNeeded);
        } finally {
            mIsDrawing = false;
            Trace.traceEnd(Trace.TRACE_TAG_VIEW);
        }
        ...
        pendingDrawFinished();
        ...
    }
```
这里主要有两个步骤，其中`pendingDrawFinished()`用来通知 wms 控件绘制结束，wms收到消息后就会把窗口的surface显示出来。
```Java
    void pendingDrawFinished() {
        if (mDrawsNeededToReport == 0) {
            throw new RuntimeException("Unbalanced drawPending/pendingDrawFinished calls");
        }
        mDrawsNeededToReport--;
        if (mDrawsNeededToReport == 0) {
            reportDrawFinished();
        }
    }

    private void reportDrawFinished() {
        try {
            mDrawsNeededToReport = 0;
            mWindowSession.finishDrawing(mWindow);
        } catch (RemoteException e) {
            // Have fun!
        }
    }
}
```
另一个则是 控件的绘制过程`draw(fullRedrawNeeded);`：
```Java
private void draw(boolean fullRedrawNeeded) {
        Surface surface = mSurface;
        ... // 控件滚动的相关计算

        final Rect dirty = mDirty;
        if (mSurfaceHolder != null) {
            // The app owns the surface, we won't draw.
            dirty.setEmpty();
            if (animating && mScroller != null) {
                mScroller.abortAnimation();
            }
            return;
        }

        if (fullRedrawNeeded) {
            mAttachInfo.mIgnoreDirtyState = true;
            dirty.set(0, 0, (int) (mWidth * appScale + 0.5f), (int) (mHeight * appScale + 0.5f));
        }
        ...
        if (!dirty.isEmpty() || mIsAnimating || accessibilityFocusDirty) {
            if (mAttachInfo.mThreadedRenderer != null && mAttachInfo.mThreadedRenderer.isEnabled()) {
               ...
            } else {
                ...
                if (!drawSoftware(surface, mAttachInfo, xOffset, yOffset, scalingRequired, dirty)) {
                    return;
                }
            }
        }
        ...
}

private boolean drawSoftware(Surface surface, AttachInfo attachInfo, int xoff, int yoff,
            boolean scalingRequired, Rect dirty) {

        // Draw with software renderer.
        final Canvas canvas;
        try {
            ...
            canvas = mSurface.lockCanvas(dirty);
            ...
        } catch (Surface.OutOfResourcesException e) {
           ...
        }

        try {
            ...
            mView.draw(canvas);
            ...
        } finally {
            surface.unlockCanvasAndPost(canvas);
            ...
        }
        return true;
    }

  public void draw(Canvas canvas) {
        final int privateFlags = mPrivateFlags;
        final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;

        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        //如果不需要绘制 渐变边界，则进入简便流程，跳过2，5步骤
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

            drawAutofilledHighlight(canvas);

            // Overlay is part of the content and draws beneath Foreground
            if (mOverlay != null && !mOverlay.isEmpty()) {



                mOverlay.getOverlayView().dispatchDraw(canvas);
            }

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // Step 7, draw the default focus highlight
            drawDefaultFocusHighlight(canvas);

            if (debugDraw()) {
                debugDrawFocus(canvas);
            }

            // we're done...
            return;
        }
        //完整流程
        ...
    }
```
draw的draw()方法，简便流程主要有以下几步：
1. 绘制背景 `drawBackground()`
2. 绘制控件自身，`onDraw(canvas)`，默认空实现
3. 绘制子控件 `dispatchDraw(canvas);`，默认空实现 
4. 绘制装饰(前景色，滚动条,etc.) `onDrawForeground(canvas);`


## 控件焦点

触摸模式和键盘模式

控件能否获取焦点的策略：
1. 当 NOT_FOCUSABLE 标记位于View.mViewFlags时，无法获取焦点
2. 当控件的父控件的DescendantFocusability 取值为 FOCUS_BLOCK_DESCENDANTS 时，无法获取焦点。
3. 当 FOCUSABLE 标记位于View.mViewFlags时，还有两种情况：
    1. 位于非触摸模式时，可以获取焦点
    2. 位于触摸模式的时候，View.mViewFlags 中存在 FUCUSABLE_IN_TOUCH_MODE标记可以获取焦点，否则不能获取焦点

获取到焦点的控件实际上只是增加了 PFLAG_FOCUSED 标记，而失去焦点则删除这个标记

## Activity和PhoneWindow

### PhoneWindow
通过 WindowManager ，ViewRootImpl 创建窗口的时候，我们仍然需要自行初始化 LayoutParams ，处理控件树的添加和删除等。 Android 在此之上又提供了一套机制，用于更简单的创建窗口和界面。而且 界面提供了预定义的样式，比如 标题栏，图标等，相比于自行创建符合Android规范的界面模板，进一步简化了开发者工作。这些工作是通过一个 com.view.Window 抽象类来实现的，目前它的唯一实现是PhoneWindow 类。我们仅仅需要通过setContentView()设置自己定义的控件树就可以得到一个带有标准模板的窗口界面，模板的样式取决于flag,theme等属性

### DecorView
DecorView 继承了 FrameLayout , 是 PhoneWindow 类里面的 控件树的根。预定义的样式就是由它实现，在我们通过setContentView()设置自己定义的View的时候，仅仅是设置View到DecorView里面成为它的子view。

## 控件的触摸事件
