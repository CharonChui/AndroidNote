View绘制过程详解
===

界面窗口中的根布局是`DecorView`，该类集成子`FrameLayout`.说到`View`绘制，想到的就是从这里入手，而`FrameLayout`集成自`ViewGroup`。感觉绘制肯定会在`ViewGroup`或者`View`中，
但是木有找到。发现`ViewGroup`实现`ViewParent`接口，而`ViewParent`有一个实现类是`ViewRootImpl`， `ViewGruop`中会使用`ViewRootImpl`...
```java
/**
 * The top of a view hierarchy, implementing the needed protocol between View
 * and the WindowManager.  This is for the most part an internal implementation
 * detail of {@link WindowManagerGlobal}.
 *
 * {@hide}
 */
@SuppressWarnings({"EmptyCatchBlock", "PointlessBooleanExpression"})
public final class ViewRootImpl implements ViewParent,
        View.AttachInfo.Callbacks, HardwareRenderer.HardwareDrawCallbacks {
		
		}
```

`View`的绘制过程从`ViewRootImpl.performTraversals()`方法开始。
首先先说明一下，这部分代码比较多，逻辑也比较麻烦，很容易弄晕，如果感觉看起来费劲，就跳过这一块，直接到下面的Measure、Layout、Draw部分开始看。
我也没有全部弄清楚。
```java
private void performTraversals() {
	// cache mView since it is used so much below...
	final View host = mView;

	if (DBG) {
		System.out.println("======================================");
		System.out.println("performTraversals");
		host.debug();
	}

	if (host == null || !mAdded)
		return;

	mIsInTraversal = true;
	mWillDrawSoon = true;
	boolean windowSizeMayChange = false;
	boolean newSurface = false;
	boolean surfaceChanged = false;
	WindowManager.LayoutParams lp = mWindowAttributes;

	int desiredWindowWidth;
	int desiredWindowHeight;

	final int viewVisibility = getHostVisibility();
	boolean viewVisibilityChanged = mViewVisibility != viewVisibility
			|| mNewSurfaceNeeded;

	WindowManager.LayoutParams params = null;
	if (mWindowAttributesChanged) {
		mWindowAttributesChanged = false;
		surfaceChanged = true;
		params = lp;
	}
	CompatibilityInfo compatibilityInfo = mDisplayAdjustments.getCompatibilityInfo();
	if (compatibilityInfo.supportsScreen() == mLastInCompatMode) {
		params = lp;
		mFullRedrawNeeded = true;
		mLayoutRequested = true;
		if (mLastInCompatMode) {
			params.privateFlags &= ~WindowManager.LayoutParams.PRIVATE_FLAG_COMPATIBLE_WINDOW;
			mLastInCompatMode = false;
		} else {
			params.privateFlags |= WindowManager.LayoutParams.PRIVATE_FLAG_COMPATIBLE_WINDOW;
			mLastInCompatMode = true;
		}
	}

	mWindowAttributesChangesFlag = 0;

	Rect frame = mWinFrame;
	if (mFirst) {
		mFullRedrawNeeded = true;
		mLayoutRequested = true;

		if (lp.type == WindowManager.LayoutParams.TYPE_STATUS_BAR_PANEL
				|| lp.type == WindowManager.LayoutParams.TYPE_INPUT_METHOD) {
			// NOTE -- system code, won't try to do compat mode.
			Point size = new Point();
			mDisplay.getRealSize(size);
			desiredWindowWidth = size.x;
			desiredWindowHeight = size.y;
		} else {
			DisplayMetrics packageMetrics =
				mView.getContext().getResources().getDisplayMetrics();
			desiredWindowWidth = packageMetrics.widthPixels;
			desiredWindowHeight = packageMetrics.heightPixels;
		}

		// We used to use the following condition to choose 32 bits drawing caches:
		// PixelFormat.hasAlpha(lp.format) || lp.format == PixelFormat.RGBX_8888
		// However, windows are now always 32 bits by default, so choose 32 bits
		mAttachInfo.mUse32BitDrawingCache = true;
		mAttachInfo.mHasWindowFocus = false;
		mAttachInfo.mWindowVisibility = viewVisibility;
		mAttachInfo.mRecomputeGlobalAttributes = false;
		viewVisibilityChanged = false;
		mLastConfiguration.setTo(host.getResources().getConfiguration());
		mLastSystemUiVisibility = mAttachInfo.mSystemUiVisibility;
		// Set the layout direction if it has not been set before (inherit is the default)
		if (mViewLayoutDirectionInitial == View.LAYOUT_DIRECTION_INHERIT) {
			host.setLayoutDirection(mLastConfiguration.getLayoutDirection());
		}
		host.dispatchAttachedToWindow(mAttachInfo, 0);
		mAttachInfo.mTreeObserver.dispatchOnWindowAttachedChange(true);
		dispatchApplyInsets(host);
		//Log.i(TAG, "Screen on initialized: " + attachInfo.mKeepScreenOn);

	} else {
		desiredWindowWidth = frame.width();
		desiredWindowHeight = frame.height();
		if (desiredWindowWidth != mWidth || desiredWindowHeight != mHeight) {
			if (DEBUG_ORIENTATION) Log.v(TAG,
					"View " + host + " resized to: " + frame);
			mFullRedrawNeeded = true;
			mLayoutRequested = true;
			windowSizeMayChange = true;
		}
	}

	if (viewVisibilityChanged) {
		mAttachInfo.mWindowVisibility = viewVisibility;
		host.dispatchWindowVisibilityChanged(viewVisibility);
		if (viewVisibility != View.VISIBLE || mNewSurfaceNeeded) {
			destroyHardwareResources();
		}
		if (viewVisibility == View.GONE) {
			// After making a window gone, we will count it as being
			// shown for the first time the next time it gets focus.
			mHasHadWindowFocus = false;
		}
	}

	// Execute enqueued actions on every traversal in case a detached view enqueued an action
	getRunQueue().executeActions(mAttachInfo.mHandler);

	boolean insetsChanged = false;

	boolean layoutRequested = mLayoutRequested && !mStopped;
	if (layoutRequested) {

		final Resources res = mView.getContext().getResources();

		if (mFirst) {
			// make sure touch mode code executes by setting cached value
			// to opposite of the added touch mode.
			mAttachInfo.mInTouchMode = !mAddedTouchMode;
			ensureTouchModeLocally(mAddedTouchMode);
		} else {
			if (!mPendingOverscanInsets.equals(mAttachInfo.mOverscanInsets)) {
				insetsChanged = true;
			}
			if (!mPendingContentInsets.equals(mAttachInfo.mContentInsets)) {
				insetsChanged = true;
			}
			if (!mPendingStableInsets.equals(mAttachInfo.mStableInsets)) {
				insetsChanged = true;
			}
			if (!mPendingVisibleInsets.equals(mAttachInfo.mVisibleInsets)) {
				mAttachInfo.mVisibleInsets.set(mPendingVisibleInsets);
				if (DEBUG_LAYOUT) Log.v(TAG, "Visible insets changing to: "
						+ mAttachInfo.mVisibleInsets);
			}
			if (lp.width == ViewGroup.LayoutParams.WRAP_CONTENT
					|| lp.height == ViewGroup.LayoutParams.WRAP_CONTENT) {
				windowSizeMayChange = true;

				if (lp.type == WindowManager.LayoutParams.TYPE_STATUS_BAR_PANEL
						|| lp.type == WindowManager.LayoutParams.TYPE_INPUT_METHOD) {
					// NOTE -- system code, won't try to do compat mode.
					Point size = new Point();
					mDisplay.getRealSize(size);
					desiredWindowWidth = size.x;
					desiredWindowHeight = size.y;
				} else {
					DisplayMetrics packageMetrics = res.getDisplayMetrics();
					desiredWindowWidth = packageMetrics.widthPixels;
					desiredWindowHeight = packageMetrics.heightPixels;
				}
			}
		}

		// Ask host how big it wants to be
		windowSizeMayChange |= measureHierarchy(host, lp, res,
				desiredWindowWidth, desiredWindowHeight);
	}

	if (collectViewAttributes()) {
		params = lp;
	}
	if (mAttachInfo.mForceReportNewAttributes) {
		mAttachInfo.mForceReportNewAttributes = false;
		params = lp;
	}

	if (mFirst || mAttachInfo.mViewVisibilityChanged) {
		mAttachInfo.mViewVisibilityChanged = false;
		int resizeMode = mSoftInputMode &
				WindowManager.LayoutParams.SOFT_INPUT_MASK_ADJUST;
		// If we are in auto resize mode, then we need to determine
		// what mode to use now.
		if (resizeMode == WindowManager.LayoutParams.SOFT_INPUT_ADJUST_UNSPECIFIED) {
			final int N = mAttachInfo.mScrollContainers.size();
			for (int i=0; i<N; i++) {
				if (mAttachInfo.mScrollContainers.get(i).isShown()) {
					resizeMode = WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE;
				}
			}
			if (resizeMode == 0) {
				resizeMode = WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN;
			}
			if ((lp.softInputMode &
					WindowManager.LayoutParams.SOFT_INPUT_MASK_ADJUST) != resizeMode) {
				lp.softInputMode = (lp.softInputMode &
						~WindowManager.LayoutParams.SOFT_INPUT_MASK_ADJUST) |
						resizeMode;
				params = lp;
			}
		}
	}

	if (params != null) {
		if ((host.mPrivateFlags & View.PFLAG_REQUEST_TRANSPARENT_REGIONS) != 0) {
			if (!PixelFormat.formatHasAlpha(params.format)) {
				params.format = PixelFormat.TRANSLUCENT;
			}
		}
		mAttachInfo.mOverscanRequested = (params.flags
				& WindowManager.LayoutParams.FLAG_LAYOUT_IN_OVERSCAN) != 0;
	}

	if (mApplyInsetsRequested) {
		mApplyInsetsRequested = false;
		mLastOverscanRequested = mAttachInfo.mOverscanRequested;
		dispatchApplyInsets(host);
		if (mLayoutRequested) {
			// Short-circuit catching a new layout request here, so
			// we don't need to go through two layout passes when things
			// change due to fitting system windows, which can happen a lot.
			windowSizeMayChange |= measureHierarchy(host, lp,
					mView.getContext().getResources(),
					desiredWindowWidth, desiredWindowHeight);
		}
	}

	if (layoutRequested) {
		// Clear this now, so that if anything requests a layout in the
		// rest of this function we will catch it and re-run a full
		// layout pass.
		mLayoutRequested = false;
	}

	boolean windowShouldResize = layoutRequested && windowSizeMayChange
		&& ((mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight())
			|| (lp.width == ViewGroup.LayoutParams.WRAP_CONTENT &&
					frame.width() < desiredWindowWidth && frame.width() != mWidth)
			|| (lp.height == ViewGroup.LayoutParams.WRAP_CONTENT &&
					frame.height() < desiredWindowHeight && frame.height() != mHeight));

	// Determine whether to compute insets.
	// If there are no inset listeners remaining then we may still need to compute
	// insets in case the old insets were non-empty and must be reset.
	final boolean computesInternalInsets =
			mAttachInfo.mTreeObserver.hasComputeInternalInsetsListeners()
			|| mAttachInfo.mHasNonEmptyGivenInternalInsets;

	boolean insetsPending = false;
	int relayoutResult = 0;

	if (mFirst || windowShouldResize || insetsChanged ||
			viewVisibilityChanged || params != null) {

		if (viewVisibility == View.VISIBLE) {
			// If this window is giving internal insets to the window
			// manager, and it is being added or changing its visibility,
			// then we want to first give the window manager "fake"
			// insets to cause it to effectively ignore the content of
			// the window during layout.  This avoids it briefly causing
			// other windows to resize/move based on the raw frame of the
			// window, waiting until we can finish laying out this window
			// and get back to the window manager with the ultimately
			// computed insets.
			insetsPending = computesInternalInsets && (mFirst || viewVisibilityChanged);
		}

		if (mSurfaceHolder != null) {
			mSurfaceHolder.mSurfaceLock.lock();
			mDrawingAllowed = true;
		}

		boolean hwInitialized = false;
		boolean contentInsetsChanged = false;
		boolean hadSurface = mSurface.isValid();

		try {
			if (DEBUG_LAYOUT) {
				Log.i(TAG, "host=w:" + host.getMeasuredWidth() + ", h:" +
						host.getMeasuredHeight() + ", params=" + params);
			}

			if (mAttachInfo.mHardwareRenderer != null) {
				// relayoutWindow may decide to destroy mSurface. As that decision
				// happens in WindowManager service, we need to be defensive here
				// and stop using the surface in case it gets destroyed.
				mAttachInfo.mHardwareRenderer.pauseSurface(mSurface);
			}
			final int surfaceGenerationId = mSurface.getGenerationId();
			relayoutResult = relayoutWindow(params, viewVisibility, insetsPending);
			if (!mDrawDuringWindowsAnimating &&
					(relayoutResult & WindowManagerGlobal.RELAYOUT_RES_ANIMATING) != 0) {
				mWindowsAnimating = true;
			}

			if (DEBUG_LAYOUT) Log.v(TAG, "relayout: frame=" + frame.toShortString()
					+ " overscan=" + mPendingOverscanInsets.toShortString()
					+ " content=" + mPendingContentInsets.toShortString()
					+ " visible=" + mPendingVisibleInsets.toShortString()
					+ " visible=" + mPendingStableInsets.toShortString()
					+ " surface=" + mSurface);

			if (mPendingConfiguration.seq != 0) {
				if (DEBUG_CONFIGURATION) Log.v(TAG, "Visible with new config: "
						+ mPendingConfiguration);
				updateConfiguration(mPendingConfiguration, !mFirst);
				mPendingConfiguration.seq = 0;
			}

			final boolean overscanInsetsChanged = !mPendingOverscanInsets.equals(
					mAttachInfo.mOverscanInsets);
			contentInsetsChanged = !mPendingContentInsets.equals(
					mAttachInfo.mContentInsets);
			final boolean visibleInsetsChanged = !mPendingVisibleInsets.equals(
					mAttachInfo.mVisibleInsets);
			final boolean stableInsetsChanged = !mPendingStableInsets.equals(
					mAttachInfo.mStableInsets);
			if (contentInsetsChanged) {
				if (mWidth > 0 && mHeight > 0 && lp != null &&
						((lp.systemUiVisibility|lp.subtreeSystemUiVisibility)
								& View.SYSTEM_UI_LAYOUT_FLAGS) == 0 &&
						mSurface != null && mSurface.isValid() &&
						!mAttachInfo.mTurnOffWindowResizeAnim &&
						mAttachInfo.mHardwareRenderer != null &&
						mAttachInfo.mHardwareRenderer.isEnabled() &&
						lp != null && !PixelFormat.formatHasAlpha(lp.format)
						&& !mBlockResizeBuffer) {

					disposeResizeBuffer();

// TODO: Again....
//                        if (mResizeBuffer == null) {
//                            mResizeBuffer = mAttachInfo.mHardwareRenderer.createDisplayListLayer(
//                                    mWidth, mHeight);
//                        }
//                        mResizeBuffer.prepare(mWidth, mHeight, false);
//                        RenderNode layerRenderNode = mResizeBuffer.startRecording();
//                        HardwareCanvas layerCanvas = layerRenderNode.start(mWidth, mHeight);
//                        try {
//                            final int restoreCount = layerCanvas.save();
//
//                            int yoff;
//                            final boolean scrolling = mScroller != null
//                                    && mScroller.computeScrollOffset();
//                            if (scrolling) {
//                                yoff = mScroller.getCurrY();
//                                mScroller.abortAnimation();
//                            } else {
//                                yoff = mScrollY;
//                            }
//
//                            layerCanvas.translate(0, -yoff);
//                            if (mTranslator != null) {
//                                mTranslator.translateCanvas(layerCanvas);
//                            }
//
//                            RenderNode renderNode = mView.mRenderNode;
//                            if (renderNode != null && renderNode.isValid()) {
//                                layerCanvas.drawDisplayList(renderNode, null,
//                                        RenderNode.FLAG_CLIP_CHILDREN);
//                            } else {
//                                mView.draw(layerCanvas);
//                            }
//
//                            drawAccessibilityFocusedDrawableIfNeeded(layerCanvas);
//
//                            mResizeBufferStartTime = SystemClock.uptimeMillis();
//                            mResizeBufferDuration = mView.getResources().getInteger(
//                                    com.android.internal.R.integer.config_mediumAnimTime);
//
//                            layerCanvas.restoreToCount(restoreCount);
//                            layerRenderNode.end(layerCanvas);
//                            layerRenderNode.setCaching(true);
//                            layerRenderNode.setLeftTopRightBottom(0, 0, mWidth, mHeight);
//                            mTempRect.set(0, 0, mWidth, mHeight);
//                        } finally {
//                            mResizeBuffer.endRecording(mTempRect);
//                        }
//                        mAttachInfo.mHardwareRenderer.flushLayerUpdates();
				}
				mAttachInfo.mContentInsets.set(mPendingContentInsets);
				if (DEBUG_LAYOUT) Log.v(TAG, "Content insets changing to: "
						+ mAttachInfo.mContentInsets);
			}
			if (overscanInsetsChanged) {
				mAttachInfo.mOverscanInsets.set(mPendingOverscanInsets);
				if (DEBUG_LAYOUT) Log.v(TAG, "Overscan insets changing to: "
						+ mAttachInfo.mOverscanInsets);
				// Need to relayout with content insets.
				contentInsetsChanged = true;
			}
			if (stableInsetsChanged) {
				mAttachInfo.mStableInsets.set(mPendingStableInsets);
				if (DEBUG_LAYOUT) Log.v(TAG, "Decor insets changing to: "
						+ mAttachInfo.mStableInsets);
				// Need to relayout with content insets.
				contentInsetsChanged = true;
			}
			if (contentInsetsChanged || mLastSystemUiVisibility !=
					mAttachInfo.mSystemUiVisibility || mApplyInsetsRequested
					|| mLastOverscanRequested != mAttachInfo.mOverscanRequested) {
				mLastSystemUiVisibility = mAttachInfo.mSystemUiVisibility;
				mLastOverscanRequested = mAttachInfo.mOverscanRequested;
				mApplyInsetsRequested = false;
				dispatchApplyInsets(host);
			}
			if (visibleInsetsChanged) {
				mAttachInfo.mVisibleInsets.set(mPendingVisibleInsets);
				if (DEBUG_LAYOUT) Log.v(TAG, "Visible insets changing to: "
						+ mAttachInfo.mVisibleInsets);
			}

			if (!hadSurface) {
				if (mSurface.isValid()) {
					// If we are creating a new surface, then we need to
					// completely redraw it.  Also, when we get to the
					// point of drawing it we will hold off and schedule
					// a new traversal instead.  This is so we can tell the
					// window manager about all of the windows being displayed
					// before actually drawing them, so it can display then
					// all at once.
					newSurface = true;
					mFullRedrawNeeded = true;
					mPreviousTransparentRegion.setEmpty();

					if (mAttachInfo.mHardwareRenderer != null) {
						try {
							hwInitialized = mAttachInfo.mHardwareRenderer.initialize(
									mSurface);
						} catch (OutOfResourcesException e) {
							handleOutOfResourcesException(e);
							return;
						}
					}
				}
			} else if (!mSurface.isValid()) {
				// If the surface has been removed, then reset the scroll
				// positions.
				if (mLastScrolledFocus != null) {
					mLastScrolledFocus.clear();
				}
				mScrollY = mCurScrollY = 0;
				if (mScroller != null) {
					mScroller.abortAnimation();
				}
				disposeResizeBuffer();
				// Our surface is gone
				if (mAttachInfo.mHardwareRenderer != null &&
						mAttachInfo.mHardwareRenderer.isEnabled()) {
					mAttachInfo.mHardwareRenderer.destroy();
				}
			} else if (surfaceGenerationId != mSurface.getGenerationId() &&
					mSurfaceHolder == null && mAttachInfo.mHardwareRenderer != null) {
				mFullRedrawNeeded = true;
				try {
					mAttachInfo.mHardwareRenderer.updateSurface(mSurface);
				} catch (OutOfResourcesException e) {
					handleOutOfResourcesException(e);
					return;
				}
			}
		} catch (RemoteException e) {
		}

		if (DEBUG_ORIENTATION) Log.v(
				TAG, "Relayout returned: frame=" + frame + ", surface=" + mSurface);

		mAttachInfo.mWindowLeft = frame.left;
		mAttachInfo.mWindowTop = frame.top;

		// !!FIXME!! This next section handles the case where we did not get the
		// window size we asked for. We should avoid this by getting a maximum size from
		// the window session beforehand.
		if (mWidth != frame.width() || mHeight != frame.height()) {
			mWidth = frame.width();
			mHeight = frame.height();
		}

		if (mSurfaceHolder != null) {
			// The app owns the surface; tell it about what is going on.
			if (mSurface.isValid()) {
				// XXX .copyFrom() doesn't work!
				//mSurfaceHolder.mSurface.copyFrom(mSurface);
				mSurfaceHolder.mSurface = mSurface;
			}
			mSurfaceHolder.setSurfaceFrameSize(mWidth, mHeight);
			mSurfaceHolder.mSurfaceLock.unlock();
			if (mSurface.isValid()) {
				if (!hadSurface) {
					mSurfaceHolder.ungetCallbacks();

					mIsCreating = true;
					mSurfaceHolderCallback.surfaceCreated(mSurfaceHolder);
					SurfaceHolder.Callback callbacks[] = mSurfaceHolder.getCallbacks();
					if (callbacks != null) {
						for (SurfaceHolder.Callback c : callbacks) {
							c.surfaceCreated(mSurfaceHolder);
						}
					}
					surfaceChanged = true;
				}
				if (surfaceChanged) {
					mSurfaceHolderCallback.surfaceChanged(mSurfaceHolder,
							lp.format, mWidth, mHeight);
					SurfaceHolder.Callback callbacks[] = mSurfaceHolder.getCallbacks();
					if (callbacks != null) {
						for (SurfaceHolder.Callback c : callbacks) {
							c.surfaceChanged(mSurfaceHolder, lp.format,
									mWidth, mHeight);
						}
					}
				}
				mIsCreating = false;
			} else if (hadSurface) {
				mSurfaceHolder.ungetCallbacks();
				SurfaceHolder.Callback callbacks[] = mSurfaceHolder.getCallbacks();
				mSurfaceHolderCallback.surfaceDestroyed(mSurfaceHolder);
				if (callbacks != null) {
					for (SurfaceHolder.Callback c : callbacks) {
						c.surfaceDestroyed(mSurfaceHolder);
					}
				}
				mSurfaceHolder.mSurfaceLock.lock();
				try {
					mSurfaceHolder.mSurface = new Surface();
				} finally {
					mSurfaceHolder.mSurfaceLock.unlock();
				}
			}
		}

		if (mAttachInfo.mHardwareRenderer != null &&
				mAttachInfo.mHardwareRenderer.isEnabled()) {
			if (hwInitialized ||
					mWidth != mAttachInfo.mHardwareRenderer.getWidth() ||
					mHeight != mAttachInfo.mHardwareRenderer.getHeight()) {
				final Rect surfaceInsets = params != null ? params.surfaceInsets : null;
				mAttachInfo.mHardwareRenderer.setup(mWidth, mHeight, surfaceInsets);
				if (!hwInitialized) {
					mAttachInfo.mHardwareRenderer.invalidate(mSurface);
					mFullRedrawNeeded = true;
				}
			}
		}

	    // 是否需要Measure
		if (!mStopped) {
			boolean focusChangedDueToTouchMode = ensureTouchModeLocally(
					(relayoutResult&WindowManagerGlobal.RELAYOUT_RES_IN_TOUCH_MODE) != 0);
			if (focusChangedDueToTouchMode || mWidth != host.getMeasuredWidth()
					|| mHeight != host.getMeasuredHeight() || contentInsetsChanged) {
				int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
				int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);

				if (DEBUG_LAYOUT) Log.v(TAG, "Ooops, something changed!  mWidth="
						+ mWidth + " measuredWidth=" + host.getMeasuredWidth()
						+ " mHeight=" + mHeight
						+ " measuredHeight=" + host.getMeasuredHeight()
						+ " coveredInsetsChanged=" + contentInsetsChanged);

				 // Ask host how big it wants to be
				performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);

				// Implementation of weights from WindowManager.LayoutParams
				// We just grow the dimensions as needed and re-measure if
				// needs be
				int width = host.getMeasuredWidth();
				int height = host.getMeasuredHeight();
				boolean measureAgain = false;

				if (lp.horizontalWeight > 0.0f) {
					width += (int) ((mWidth - width) * lp.horizontalWeight);
					childWidthMeasureSpec = MeasureSpec.makeMeasureSpec(width,
							MeasureSpec.EXACTLY);
					measureAgain = true;
				}
				if (lp.verticalWeight > 0.0f) {
					height += (int) ((mHeight - height) * lp.verticalWeight);
					childHeightMeasureSpec = MeasureSpec.makeMeasureSpec(height,
							MeasureSpec.EXACTLY);
					measureAgain = true;
				}

				if (measureAgain) {
					if (DEBUG_LAYOUT) Log.v(TAG,
							"And hey let's measure once more: width=" + width
							+ " height=" + height);
					performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
				}

				layoutRequested = true;
			}
		}
	} else {
		// Not the first pass and no window/insets/visibility change but the window
		// may have moved and we need check that and if so to update the left and right
		// in the attach info. We translate only the window frame since on window move
		// the window manager tells us only for the new frame but the insets are the
		// same and we do not want to translate them more than once.

		// TODO: Well, we are checking whether the frame has changed similarly
		// to how this is done for the insets. This is however incorrect since
		// the insets and the frame are translated. For example, the old frame
		// was (1, 1 - 1, 1) and was translated to say (2, 2 - 2, 2), now the new
		// reported frame is (2, 2 - 2, 2) which implies no change but this is not
		// true since we are comparing a not translated value to a translated one.
		// This scenario is rare but we may want to fix that.

		final boolean windowMoved = (mAttachInfo.mWindowLeft != frame.left
				|| mAttachInfo.mWindowTop != frame.top);
		if (windowMoved) {
			if (mTranslator != null) {
				mTranslator.translateRectInScreenToAppWinFrame(frame);
			}
			mAttachInfo.mWindowLeft = frame.left;
			mAttachInfo.mWindowTop = frame.top;
		}
	}

	final boolean didLayout = layoutRequested && !mStopped;
	boolean triggerGlobalLayoutListener = didLayout
			|| mAttachInfo.mRecomputeGlobalAttributes;
	// 是否需要Layout
	if (didLayout) {
		performLayout(lp, desiredWindowWidth, desiredWindowHeight);

		// By this point all views have been sized and positioned
		// We can compute the transparent area

		if ((host.mPrivateFlags & View.PFLAG_REQUEST_TRANSPARENT_REGIONS) != 0) {
			// start out transparent
			// TODO: AVOID THAT CALL BY CACHING THE RESULT?
			host.getLocationInWindow(mTmpLocation);
			mTransparentRegion.set(mTmpLocation[0], mTmpLocation[1],
					mTmpLocation[0] + host.mRight - host.mLeft,
					mTmpLocation[1] + host.mBottom - host.mTop);

			host.gatherTransparentRegion(mTransparentRegion);
			if (mTranslator != null) {
				mTranslator.translateRegionInWindowToScreen(mTransparentRegion);
			}

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

		if (DBG) {
			System.out.println("======================================");
			System.out.println("performTraversals -- after setFrame");
			host.debug();
		}
	}

	if (triggerGlobalLayoutListener) {
		mAttachInfo.mRecomputeGlobalAttributes = false;
		mAttachInfo.mTreeObserver.dispatchOnGlobalLayout();
	}

	if (computesInternalInsets) {
		// Clear the original insets.
		final ViewTreeObserver.InternalInsetsInfo insets = mAttachInfo.mGivenInternalInsets;
		insets.reset();

		// Compute new insets in place.
		mAttachInfo.mTreeObserver.dispatchOnComputeInternalInsets(insets);
		mAttachInfo.mHasNonEmptyGivenInternalInsets = !insets.isEmpty();

		// Tell the window manager.
		if (insetsPending || !mLastGivenInsets.equals(insets)) {
			mLastGivenInsets.set(insets);

			// Translate insets to screen coordinates if needed.
			final Rect contentInsets;
			final Rect visibleInsets;
			final Region touchableRegion;
			if (mTranslator != null) {
				contentInsets = mTranslator.getTranslatedContentInsets(insets.contentInsets);
				visibleInsets = mTranslator.getTranslatedVisibleInsets(insets.visibleInsets);
				touchableRegion = mTranslator.getTranslatedTouchableArea(insets.touchableRegion);
			} else {
				contentInsets = insets.contentInsets;
				visibleInsets = insets.visibleInsets;
				touchableRegion = insets.touchableRegion;
			}

			try {
				mWindowSession.setInsets(mWindow, insets.mTouchableInsets,
						contentInsets, visibleInsets, touchableRegion);
			} catch (RemoteException e) {
			}
		}
	}

	boolean skipDraw = false;

	if (mFirst) {
		// handle first focus request
		if (DEBUG_INPUT_RESIZE) Log.v(TAG, "First: mView.hasFocus()="
				+ mView.hasFocus());
		if (mView != null) {
			if (!mView.hasFocus()) {
				mView.requestFocus(View.FOCUS_FORWARD);
				if (DEBUG_INPUT_RESIZE) Log.v(TAG, "First: requested focused view="
						+ mView.findFocus());
			} else {
				if (DEBUG_INPUT_RESIZE) Log.v(TAG, "First: existing focused view="
						+ mView.findFocus());
			}
		}
		if ((relayoutResult & WindowManagerGlobal.RELAYOUT_RES_ANIMATING) != 0) {
			// The first time we relayout the window, if the system is
			// doing window animations, we want to hold of on any future
			// draws until the animation is done.
			mWindowsAnimating = true;
		}
	} else if (mWindowsAnimating) {
		skipDraw = true;
	}

	mFirst = false;
	mWillDrawSoon = false;
	mNewSurfaceNeeded = false;
	mViewVisibility = viewVisibility;

	if (mAttachInfo.mHasWindowFocus && !isInLocalFocusMode()) {
		final boolean imTarget = WindowManager.LayoutParams
				.mayUseInputMethod(mWindowAttributes.flags);
		if (imTarget != mLastWasImTarget) {
			mLastWasImTarget = imTarget;
			InputMethodManager imm = InputMethodManager.peekInstance();
			if (imm != null && imTarget) {
				imm.startGettingWindowFocus(mView);
				imm.onWindowFocus(mView, mView.findFocus(),
						mWindowAttributes.softInputMode,
						!mHasHadWindowFocus, mWindowAttributes.flags);
			}
		}
	}

	// Remember if we must report the next draw.
	if ((relayoutResult & WindowManagerGlobal.RELAYOUT_RES_FIRST_TIME) != 0) {
		mReportNextDraw = true;
	}

	boolean cancelDraw = mAttachInfo.mTreeObserver.dispatchOnPreDraw() ||
			viewVisibility != View.VISIBLE;

	// 是否需要Draw
	if (!cancelDraw && !newSurface) {
		if (!skipDraw || mReportNextDraw) {
			if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
				for (int i = 0; i < mPendingTransitions.size(); ++i) {
					mPendingTransitions.get(i).startChangingAnimations();
				}
				mPendingTransitions.clear();
			}

			performDraw();
		}
	} else {
		if (viewVisibility == View.VISIBLE) {
			// Try again
			scheduleTraversals();
		} else if (mPendingTransitions != null && mPendingTransitions.size() > 0) {
			for (int i = 0; i < mPendingTransitions.size(); ++i) {
				mPendingTransitions.get(i).endChangingAnimations();
			}
			mPendingTransitions.clear();
		}
	}

	mIsInTraversal = false;
}
```

从上面源码可以看出,`performTraversals()`方法中会依次做三件事：
- `performMeasure()`, 内部是` mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);`
- `performLayout()`, 内部是`mView.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());`
- `performDraw()`, 内部是`draw(fullRedrawNeeded);`

至此`View`绘制的三个过程已经展现：

`Measure`
=====

在`performMeasure()`方法中会调用`View.measure()`方法， 源码如下：
```java
/**
 * <p>
 * This is called to find out how big a view should be. The parent
 * supplies constraint information in the width and height parameters.
 * </p>
 *
 * <p>
 * The actual measurement work of a view is performed in
 * {@link #onMeasure(int, int)}, called by this method. Therefore, only
 * {@link #onMeasure(int, int)} can and must be overridden by subclasses.
 * </p>
 *
 *
 * @param widthMeasureSpec Horizontal space requirements as imposed by the
 *        parent
 * @param heightMeasureSpec Vertical space requirements as imposed by the
 *        parent
 *
 * @see #onMeasure(int, int)
 */
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
	boolean optical = isLayoutModeOptical(this);
	if (optical != isLayoutModeOptical(mParent)) {
		Insets insets = getOpticalInsets();
		int oWidth  = insets.left + insets.right;
		int oHeight = insets.top  + insets.bottom;
		widthMeasureSpec  = MeasureSpec.adjust(widthMeasureSpec,  optical ? -oWidth  : oWidth);
		heightMeasureSpec = MeasureSpec.adjust(heightMeasureSpec, optical ? -oHeight : oHeight);
	}

	// Suppress sign extension for the low bytes
	long key = (long) widthMeasureSpec << 32 | (long) heightMeasureSpec & 0xffffffffL;
	if (mMeasureCache == null) mMeasureCache = new LongSparseLongArray(2);

	if ((mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ||
			widthMeasureSpec != mOldWidthMeasureSpec ||
			heightMeasureSpec != mOldHeightMeasureSpec) {

		// first clears the measured dimension flag
		mPrivateFlags &= ~PFLAG_MEASURED_DIMENSION_SET;

		resolveRtlPropertiesIfNeeded();

		int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
				mMeasureCache.indexOfKey(key);
		if (cacheIndex < 0 || sIgnoreMeasureCache) {
			// 调用onMeasure方法
			// measure ourselves, this should set the measured dimension flag back
			onMeasure(widthMeasureSpec, heightMeasureSpec);
			mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
		} else {
			long value = mMeasureCache.valueAt(cacheIndex);
			// Casting a long to int drops the high 32 bits, no mask needed
			setMeasuredDimensionRaw((int) (value >> 32), (int) value);
			mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
		}

		// flag not set, setMeasuredDimension() was not invoked, we raise
		// an exception to warn the developer
		if ((mPrivateFlags & PFLAG_MEASURED_DIMENSION_SET) != PFLAG_MEASURED_DIMENSION_SET) {
			throw new IllegalStateException("onMeasure() did not set the"
					+ " measured dimension by calling"
					+ " setMeasuredDimension()");
		}

		mPrivateFlags |= PFLAG_LAYOUT_REQUIRED;
	}

	mOldWidthMeasureSpec = widthMeasureSpec;
	mOldHeightMeasureSpec = heightMeasureSpec;

	mMeasureCache.put(key, ((long) mMeasuredWidth) << 32 |
			(long) mMeasuredHeight & 0xffffffffL); // suppress sign extension
}
```

在`measure`方法中会调用`onMeasure`方法。
`onMeasure()`方法的源码如下：
```java
/**
 * <p>
 * Measure the view and its content to determine the measured width and the
 * measured height. This method is invoked by {@link #measure(int, int)} and
 * should be overriden by subclasses to provide accurate and efficient
 * measurement of their contents.
 * </p>
 *
 * <p>
 * <strong>CONTRACT:</strong> When overriding this method, you
 * <em>must</em> call {@link #setMeasuredDimension(int, int)} to store the
 * measured width and height of this view. Failure to do so will trigger an
 * <code>IllegalStateException</code>, thrown by
 * {@link #measure(int, int)}. Calling the superclass'
 * {@link #onMeasure(int, int)} is a valid use.
 * </p>
 *
 * <p>
 * The base class implementation of measure defaults to the background size,
 * unless a larger size is allowed by the MeasureSpec. Subclasses should
 * override {@link #onMeasure(int, int)} to provide better measurements of
 * their content.
 * </p>
 *
 * <p>
 * If this method is overridden, it is the subclass's responsibility to make
 * sure the measured height and width are at least the view's minimum height
 * and width ({@link #getSuggestedMinimumHeight()} and
 * {@link #getSuggestedMinimumWidth()}).
 * </p>
 *
 * @param widthMeasureSpec horizontal space requirements as imposed by the parent.
 *                         The requirements are encoded with
 *                         {@link android.view.View.MeasureSpec}.
 * @param heightMeasureSpec vertical space requirements as imposed by the parent.
 *                         The requirements are encoded with
 *                         {@link android.view.View.MeasureSpec}.
 *
 * @see #getMeasuredWidth()
 * @see #getMeasuredHeight()
 * @see #setMeasuredDimension(int, int)
 * @see #getSuggestedMinimumHeight()
 * @see #getSuggestedMinimumWidth()
 * @see android.view.View.MeasureSpec#getMode(int)
 * @see android.view.View.MeasureSpec#getSize(int)
 */
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
			getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```
`getDefaultSize()`源码如下：
```java
/**
 * Utility to return a default size. Uses the supplied size if the
 * MeasureSpec imposed no constraints. Will get larger if allowed
 * by the MeasureSpec.
 *
 * @param size Default size for this view
 * @param measureSpec Constraints imposed by the parent
 * @return The size this view should be.
 */
public static int getDefaultSize(int size, int measureSpec) {
	int result = size;
	int specMode = MeasureSpec.getMode(measureSpec);
	int specSize = MeasureSpec.getSize(measureSpec);

	switch (specMode) {
	case MeasureSpec.UNSPECIFIED:
		result = size;
		break;
	case MeasureSpec.AT_MOST:
	case MeasureSpec.EXACTLY:
		result = specSize;
		break;
	}
	return result;
}
```

`setMeasuredDimension()`方法如下： 
```java
/**
 * <p>This method must be called by {@link #onMeasure(int, int)} to store the
 * measured width and measured height. Failing to do so will trigger an
 * exception at measurement time.</p>
 *
 * @param measuredWidth The measured width of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 * @param measuredHeight The measured height of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 */
protected final void setMeasuredDimension(int measuredWidth, int measuredHeight) {
	boolean optical = isLayoutModeOptical(this);
	if (optical != isLayoutModeOptical(mParent)) {
		Insets insets = getOpticalInsets();
		int opticalWidth  = insets.left + insets.right;
		int opticalHeight = insets.top  + insets.bottom;

		measuredWidth  += optical ? opticalWidth  : -opticalWidth;
		measuredHeight += optical ? opticalHeight : -opticalHeight;
	}
	setMeasuredDimensionRaw(measuredWidth, measuredHeight);
}
```
`setMeasuredDimensionRaw()`方法如下： 
```java
/**
 * Sets the measured dimension without extra processing for things like optical bounds.
 * Useful for reapplying consistent values that have already been cooked with adjustments
 * for optical bounds, etc. such as those from the measurement cache.
 *
 * @param measuredWidth The measured width of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 * @param measuredHeight The measured height of this view.  May be a complex
 * bit mask as defined by {@link #MEASURED_SIZE_MASK} and
 * {@link #MEASURED_STATE_TOO_SMALL}.
 */
private void setMeasuredDimensionRaw(int measuredWidth, int measuredHeight) {
	mMeasuredWidth = measuredWidth;
	mMeasuredHeight = measuredHeight;

	mPrivateFlags |= PFLAG_MEASURED_DIMENSION_SET;
}
```



`Layout`
=====


`Draw`
=====


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 