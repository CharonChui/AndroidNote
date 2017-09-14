Activity界面绘制过程详解
===

设置界面首先就是`Activity.setContentView()`方法：我们先看一下他的源码：
```java
/**
 * Set the activity content from a layout resource.  The resource will be
 * inflated, adding all top-level views to the activity.
 *
 * @param layoutResID Resource ID to be inflated.
 *
 * @see #setContentView(android.view.View)
 * @see #setContentView(android.view.View, android.view.ViewGroup.LayoutParams)
 */
public void setContentView(int layoutResID) {
	getWindow().setContentView(layoutResID);
	initWindowDecorActionBar();
}
```
`getWindow()`就是获取该页面的窗体`Window`，该类是一个抽象类，他有一个唯一的子类`PhoneWindow`.
```java
/**
 * Retrieve the current {@link android.view.Window} for the activity.
 * This can be used to directly access parts of the Window API that
 * are not available through Activity/Screen.
 *
 * @return Window The current window, or null if the activity is not
 *         visual.
 */
public Window getWindow() {
	return mWindow;
}
```
接下来，我们看一下`PhoneWindow.setContentView()`方法。(简单的理解是`PhoneWindow`把`DectorView`(`FrameLayout`的子类)进行包装，将它作为窗口的根`View`)
```java
@Override
public void setContentView(int layoutResID) {

	// mContentParent 是当前window显示内容的父布局，它只能是mDecor或mDecor的子View,如果mContentParent为null，说明是第一次调用setContentView方法
	// Note: FEATURE_CONTENT_TRANSITIONS may be set in the process of installing the window
	// decor, when theme attributes and the like are crystalized. Do not check the feature
	// before this happens.
	if (mContentParent == null) {
	    // 内部会创建mContentParent, 如果mDecor为null就创建，然后如果mContentParent为null，就根据当前要显示的主题去添加对应的布局，
		// 并把该布局中id为content的ViewGroup赋值给mContentParent。
		installDecor();
	} else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
		// 多次调用setContentView，可能是第二次、第三次，先把之前的页面内容移除
		mContentParent.removeAllViews();
	}

	if (hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
		final Scene newScene = Scene.getSceneForLayout(mContentParent, layoutResID,
				getContext());
		transitionTo(newScene);
	} else {
	    // 把布局inflate后添加到mContentParent中
		mLayoutInflater.inflate(layoutResID, mContentParent);
	}
	final Callback cb = getCallback();
	if (cb != null && !isDestroyed()) {
		cb.onContentChanged();
	}
}
```

上面简单的说明了`installDecor()`的作用，这里我们在源码中仔细说明一下, 通过这个源码
我们知道设置主题要在`setContentView()`之前去调用，如用代码设置`requestWindowFeature()`设置主题时要在`setContentView()`之前设置才有用.
```java
private void installDecor() {
	if (mDecor == null) {
		// 内部就是new 一个 DecorView
		mDecor = generateDecor();
		mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
		mDecor.setIsRootNamespace(true);
		if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
			mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
		}
	}
	if (mContentParent == null) {
		// 根据当前主题去选择对应的布局文件，然后把布局中的id=content(一般为FrameLayout)部分赋值给mContentParent.
		mContentParent = generateLayout(mDecor);

		// Set up decor part of UI to ignore fitsSystemWindows if appropriate.
		mDecor.makeOptionalFitsSystemWindows();

		final DecorContentParent decorContentParent = (DecorContentParent) mDecor.findViewById(
				R.id.decor_content_parent);

		if (decorContentParent != null) {
			mDecorContentParent = decorContentParent;
			mDecorContentParent.setWindowCallback(getCallback());
			if (mDecorContentParent.getTitle() == null) {
				mDecorContentParent.setWindowTitle(mTitle);
			}

			final int localFeatures = getLocalFeatures();
			for (int i = 0; i < FEATURE_MAX; i++) {
				if ((localFeatures & (1 << i)) != 0) {
					mDecorContentParent.initFeature(i);
				}
			}

			mDecorContentParent.setUiOptions(mUiOptions);

			if ((mResourcesSetFlags & FLAG_RESOURCE_SET_ICON) != 0 ||
					(mIconRes != 0 && !mDecorContentParent.hasIcon())) {
				mDecorContentParent.setIcon(mIconRes);
			} else if ((mResourcesSetFlags & FLAG_RESOURCE_SET_ICON) == 0 &&
					mIconRes == 0 && !mDecorContentParent.hasIcon()) {
				mDecorContentParent.setIcon(
						getContext().getPackageManager().getDefaultActivityIcon());
				mResourcesSetFlags |= FLAG_RESOURCE_SET_ICON_FALLBACK;
			}
			if ((mResourcesSetFlags & FLAG_RESOURCE_SET_LOGO) != 0 ||
					(mLogoRes != 0 && !mDecorContentParent.hasLogo())) {
				mDecorContentParent.setLogo(mLogoRes);
			}

			// Invalidate if the panel menu hasn't been created before this.
			// Panel menu invalidation is deferred avoiding application onCreateOptionsMenu
			// being called in the middle of onCreate or similar.
			// A pending invalidation will typically be resolved before the posted message
			// would run normally in order to satisfy instance state restoration.
			PanelFeatureState st = getPanelState(FEATURE_OPTIONS_PANEL, false);
			if (!isDestroyed() && (st == null || st.menu == null)) {
				invalidatePanelMenu(FEATURE_ACTION_BAR);
			}
		} else {
			mTitleView = (TextView)findViewById(R.id.title);
			if (mTitleView != null) {
				mTitleView.setLayoutDirection(mDecor.getLayoutDirection());
				if ((getLocalFeatures() & (1 << FEATURE_NO_TITLE)) != 0) {
					View titleContainer = findViewById(
							R.id.title_container);
					if (titleContainer != null) {
						titleContainer.setVisibility(View.GONE);
					} else {
						mTitleView.setVisibility(View.GONE);
					}
					if (mContentParent instanceof FrameLayout) {
						((FrameLayout)mContentParent).setForeground(null);
					}
				} else {
					mTitleView.setText(mTitle);
				}
			}
		}

		if (mDecor.getBackground() == null && mBackgroundFallbackResource != 0) {
			mDecor.setBackgroundFallback(mBackgroundFallbackResource);
		}

		// Only inflate or create a new TransitionManager if the caller hasn't
		// already set a custom one.
		if (hasFeature(FEATURE_ACTIVITY_TRANSITIONS)) {
			if (mTransitionManager == null) {
				final int transitionRes = getWindowStyle().getResourceId(
						R.styleable.Window_windowContentTransitionManager,
						0);
				if (transitionRes != 0) {
					final TransitionInflater inflater = TransitionInflater.from(getContext());
					mTransitionManager = inflater.inflateTransitionManager(transitionRes,
							mContentParent);
				} else {
					mTransitionManager = new TransitionManager();
				}
			}

			mEnterTransition = getTransition(mEnterTransition, null,
					R.styleable.Window_windowEnterTransition);
			mReturnTransition = getTransition(mReturnTransition, USE_DEFAULT_TRANSITION,
					R.styleable.Window_windowReturnTransition);
			mExitTransition = getTransition(mExitTransition, null,
					R.styleable.Window_windowExitTransition);
			mReenterTransition = getTransition(mReenterTransition, USE_DEFAULT_TRANSITION,
					R.styleable.Window_windowReenterTransition);
			mSharedElementEnterTransition = getTransition(mSharedElementEnterTransition, null,
					R.styleable.Window_windowSharedElementEnterTransition);
			mSharedElementReturnTransition = getTransition(mSharedElementReturnTransition,
					USE_DEFAULT_TRANSITION,
					R.styleable.Window_windowSharedElementReturnTransition);
			mSharedElementExitTransition = getTransition(mSharedElementExitTransition, null,
					R.styleable.Window_windowSharedElementExitTransition);
			mSharedElementReenterTransition = getTransition(mSharedElementReenterTransition,
					USE_DEFAULT_TRANSITION,
					R.styleable.Window_windowSharedElementReenterTransition);
			if (mAllowEnterTransitionOverlap == null) {
				mAllowEnterTransitionOverlap = getWindowStyle().getBoolean(
						R.styleable.Window_windowAllowEnterTransitionOverlap, true);
			}
			if (mAllowReturnTransitionOverlap == null) {
				mAllowReturnTransitionOverlap = getWindowStyle().getBoolean(
						R.styleable.Window_windowAllowReturnTransitionOverlap, true);
			}
			if (mBackgroundFadeDurationMillis < 0) {
				mBackgroundFadeDurationMillis = getWindowStyle().getInteger(
						R.styleable.Window_windowTransitionBackgroundFadeDuration,
						DEFAULT_BACKGROUND_FADE_DURATION_MS);
			}
			if (mSharedElementsUseOverlay == null) {
				mSharedElementsUseOverlay = getWindowStyle().getBoolean(
						R.styleable.Window_windowSharedElementsUseOverlay, true);
			}
		}
	}
}
```

我们先看一下`generateLayout`的源码：
```java
protected ViewGroup generateLayout(DecorView decor) {
	// Apply data from current theme.

	TypedArray a = getWindowStyle();

	if (false) {
		System.out.println("From style:");
		String s = "Attrs:";
		for (int i = 0; i < R.styleable.Window.length; i++) {
			s = s + " " + Integer.toHexString(R.styleable.Window[i]) + "="
					+ a.getString(i);
		}
		System.out.println(s);
	}

	mIsFloating = a.getBoolean(R.styleable.Window_windowIsFloating, false);
	int flagsToUpdate = (FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR)
			& (~getForcedWindowFlags());
	if (mIsFloating) {
		setLayout(WRAP_CONTENT, WRAP_CONTENT);
		setFlags(0, flagsToUpdate);
	} else {
		setFlags(FLAG_LAYOUT_IN_SCREEN|FLAG_LAYOUT_INSET_DECOR, flagsToUpdate);
	}

	if (a.getBoolean(R.styleable.Window_windowNoTitle, false)) {
		requestFeature(FEATURE_NO_TITLE);
	} else if (a.getBoolean(R.styleable.Window_windowActionBar, false)) {
		// Don't allow an action bar if there is no title.
		requestFeature(FEATURE_ACTION_BAR);
	}

	if (a.getBoolean(R.styleable.Window_windowActionBarOverlay, false)) {
		requestFeature(FEATURE_ACTION_BAR_OVERLAY);
	}

	if (a.getBoolean(R.styleable.Window_windowActionModeOverlay, false)) {
		requestFeature(FEATURE_ACTION_MODE_OVERLAY);
	}

	if (a.getBoolean(R.styleable.Window_windowSwipeToDismiss, false)) {
		requestFeature(FEATURE_SWIPE_TO_DISMISS);
	}

	if (a.getBoolean(R.styleable.Window_windowFullscreen, false)) {
		setFlags(FLAG_FULLSCREEN, FLAG_FULLSCREEN & (~getForcedWindowFlags()));
	}

	if (a.getBoolean(R.styleable.Window_windowTranslucentStatus,
			false)) {
		setFlags(FLAG_TRANSLUCENT_STATUS, FLAG_TRANSLUCENT_STATUS
				& (~getForcedWindowFlags()));
	}

	if (a.getBoolean(R.styleable.Window_windowTranslucentNavigation,
			false)) {
		setFlags(FLAG_TRANSLUCENT_NAVIGATION, FLAG_TRANSLUCENT_NAVIGATION
				& (~getForcedWindowFlags()));
	}

	if (a.getBoolean(R.styleable.Window_windowOverscan, false)) {
		setFlags(FLAG_LAYOUT_IN_OVERSCAN, FLAG_LAYOUT_IN_OVERSCAN&(~getForcedWindowFlags()));
	}

	if (a.getBoolean(R.styleable.Window_windowShowWallpaper, false)) {
		setFlags(FLAG_SHOW_WALLPAPER, FLAG_SHOW_WALLPAPER&(~getForcedWindowFlags()));
	}

	if (a.getBoolean(R.styleable.Window_windowEnableSplitTouch,
			getContext().getApplicationInfo().targetSdkVersion
					>= android.os.Build.VERSION_CODES.HONEYCOMB)) {
		setFlags(FLAG_SPLIT_TOUCH, FLAG_SPLIT_TOUCH&(~getForcedWindowFlags()));
	}

	a.getValue(R.styleable.Window_windowMinWidthMajor, mMinWidthMajor);
	a.getValue(R.styleable.Window_windowMinWidthMinor, mMinWidthMinor);
	if (a.hasValue(R.styleable.Window_windowFixedWidthMajor)) {
		if (mFixedWidthMajor == null) mFixedWidthMajor = new TypedValue();
		a.getValue(R.styleable.Window_windowFixedWidthMajor,
				mFixedWidthMajor);
	}
	if (a.hasValue(R.styleable.Window_windowFixedWidthMinor)) {
		if (mFixedWidthMinor == null) mFixedWidthMinor = new TypedValue();
		a.getValue(R.styleable.Window_windowFixedWidthMinor,
				mFixedWidthMinor);
	}
	if (a.hasValue(R.styleable.Window_windowFixedHeightMajor)) {
		if (mFixedHeightMajor == null) mFixedHeightMajor = new TypedValue();
		a.getValue(R.styleable.Window_windowFixedHeightMajor,
				mFixedHeightMajor);
	}
	if (a.hasValue(R.styleable.Window_windowFixedHeightMinor)) {
		if (mFixedHeightMinor == null) mFixedHeightMinor = new TypedValue();
		a.getValue(R.styleable.Window_windowFixedHeightMinor,
				mFixedHeightMinor);
	}
	if (a.getBoolean(R.styleable.Window_windowContentTransitions, false)) {
		requestFeature(FEATURE_CONTENT_TRANSITIONS);
	}
	if (a.getBoolean(R.styleable.Window_windowActivityTransitions, false)) {
		requestFeature(FEATURE_ACTIVITY_TRANSITIONS);
	}

	final WindowManager windowService = (WindowManager) getContext().getSystemService(
			Context.WINDOW_SERVICE);
	if (windowService != null) {
		final Display display = windowService.getDefaultDisplay();
		final boolean shouldUseBottomOutset =
				display.getDisplayId() == Display.DEFAULT_DISPLAY
						|| (getForcedWindowFlags() & FLAG_FULLSCREEN) != 0;
		if (shouldUseBottomOutset && a.hasValue(R.styleable.Window_windowOutsetBottom)) {
			if (mOutsetBottom == null) mOutsetBottom = new TypedValue();
			a.getValue(R.styleable.Window_windowOutsetBottom,
					mOutsetBottom);
		}
	}

	final Context context = getContext();
	final int targetSdk = context.getApplicationInfo().targetSdkVersion;
	final boolean targetPreHoneycomb = targetSdk < android.os.Build.VERSION_CODES.HONEYCOMB;
	final boolean targetPreIcs = targetSdk < android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH;
	final boolean targetPreL = targetSdk < android.os.Build.VERSION_CODES.LOLLIPOP;
	final boolean targetHcNeedsOptions = context.getResources().getBoolean(
			R.bool.target_honeycomb_needs_options_menu);
	final boolean noActionBar = !hasFeature(FEATURE_ACTION_BAR) || hasFeature(FEATURE_NO_TITLE);

	if (targetPreHoneycomb || (targetPreIcs && targetHcNeedsOptions && noActionBar)) {
		addFlags(WindowManager.LayoutParams.FLAG_NEEDS_MENU_KEY);
	} else {
		clearFlags(WindowManager.LayoutParams.FLAG_NEEDS_MENU_KEY);
	}

	// Non-floating windows on high end devices must put up decor beneath the system bars and
	// therefore must know about visibility changes of those.
	if (!mIsFloating && ActivityManager.isHighEndGfx()) {
		if (!targetPreL && a.getBoolean(
				R.styleable.Window_windowDrawsSystemBarBackgrounds,
				false)) {
			setFlags(FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS,
					FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS & ~getForcedWindowFlags());
		}
	}
	if (!mForcedStatusBarColor) {
		mStatusBarColor = a.getColor(R.styleable.Window_statusBarColor, 0xFF000000);
	}
	if (!mForcedNavigationBarColor) {
		mNavigationBarColor = a.getColor(R.styleable.Window_navigationBarColor, 0xFF000000);
	}

	if (mAlwaysReadCloseOnTouchAttr || getContext().getApplicationInfo().targetSdkVersion
			>= android.os.Build.VERSION_CODES.HONEYCOMB) {
		if (a.getBoolean(
				R.styleable.Window_windowCloseOnTouchOutside,
				false)) {
			setCloseOnTouchOutsideIfNotSet(true);
		}
	}

	WindowManager.LayoutParams params = getAttributes();

	if (!hasSoftInputMode()) {
		params.softInputMode = a.getInt(
				R.styleable.Window_windowSoftInputMode,
				params.softInputMode);
	}

	if (a.getBoolean(R.styleable.Window_backgroundDimEnabled,
			mIsFloating)) {
		/* All dialogs should have the window dimmed */
		if ((getForcedWindowFlags()&WindowManager.LayoutParams.FLAG_DIM_BEHIND) == 0) {
			params.flags |= WindowManager.LayoutParams.FLAG_DIM_BEHIND;
		}
		if (!haveDimAmount()) {
			params.dimAmount = a.getFloat(
					android.R.styleable.Window_backgroundDimAmount, 0.5f);
		}
	}

	if (params.windowAnimations == 0) {
		params.windowAnimations = a.getResourceId(
				R.styleable.Window_windowAnimationStyle, 0);
	}

	// The rest are only done if this window is not embedded; otherwise,
	// the values are inherited from our container.
	if (getContainer() == null) {
		if (mBackgroundDrawable == null) {
			if (mBackgroundResource == 0) {
				mBackgroundResource = a.getResourceId(
						R.styleable.Window_windowBackground, 0);
			}
			if (mFrameResource == 0) {
				mFrameResource = a.getResourceId(R.styleable.Window_windowFrame, 0);
			}
			mBackgroundFallbackResource = a.getResourceId(
					R.styleable.Window_windowBackgroundFallback, 0);
			if (false) {
				System.out.println("Background: "
						+ Integer.toHexString(mBackgroundResource) + " Frame: "
						+ Integer.toHexString(mFrameResource));
			}
		}
		mElevation = a.getDimension(R.styleable.Window_windowElevation, 0);
		mClipToOutline = a.getBoolean(R.styleable.Window_windowClipToOutline, false);
		mTextColor = a.getColor(R.styleable.Window_textColor, Color.TRANSPARENT);
	}

	// 上面的那一块都是对Activity中设置的主题进行判断。下面就是根据不同的主题，选择不同的布局文件。
	// Inflate the window decor.

	int layoutResource;
	int features = getLocalFeatures();
	// System.out.println("Features: 0x" + Integer.toHexString(features));
	if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
		layoutResource = R.layout.screen_swipe_dismiss;
	} else if ((features & ((1 << FEATURE_LEFT_ICON) | (1 << FEATURE_RIGHT_ICON))) != 0) {
		if (mIsFloating) {
			TypedValue res = new TypedValue();
			getContext().getTheme().resolveAttribute(
					R.attr.dialogTitleIconsDecorLayout, res, true);
			layoutResource = res.resourceId;
		} else {
			layoutResource = R.layout.screen_title_icons;
		}
		// XXX Remove this once action bar supports these features.
		removeFeature(FEATURE_ACTION_BAR);
		// System.out.println("Title Icons!");
	} else if ((features & ((1 << FEATURE_PROGRESS) | (1 << FEATURE_INDETERMINATE_PROGRESS))) != 0
			&& (features & (1 << FEATURE_ACTION_BAR)) == 0) {
		// Special case for a window with only a progress bar (and title).
		// XXX Need to have a no-title version of embedded windows.
		layoutResource = R.layout.screen_progress;
		// System.out.println("Progress!");
	} else if ((features & (1 << FEATURE_CUSTOM_TITLE)) != 0) {
		// Special case for a window with a custom title.
		// If the window is floating, we need a dialog layout
		if (mIsFloating) {
			TypedValue res = new TypedValue();
			getContext().getTheme().resolveAttribute(
					R.attr.dialogCustomTitleDecorLayout, res, true);
			layoutResource = res.resourceId;
		} else {
			layoutResource = R.layout.screen_custom_title;
		}
		// XXX Remove this once action bar supports these features.
		removeFeature(FEATURE_ACTION_BAR);
	} else if ((features & (1 << FEATURE_NO_TITLE)) == 0) {
		// If no other features and not embedded, only need a title.
		// If the window is floating, we need a dialog layout
		if (mIsFloating) {
			TypedValue res = new TypedValue();
			getContext().getTheme().resolveAttribute(
					R.attr.dialogTitleDecorLayout, res, true);
			layoutResource = res.resourceId;
		} else if ((features & (1 << FEATURE_ACTION_BAR)) != 0) {
			layoutResource = a.getResourceId(
					R.styleable.Window_windowActionBarFullscreenDecorLayout,
					R.layout.screen_action_bar);
		} else {
			layoutResource = R.layout.screen_title;
		}
		// System.out.println("Title!");
	} else if ((features & (1 << FEATURE_ACTION_MODE_OVERLAY)) != 0) {
		layoutResource = R.layout.screen_simple_overlay_action_mode;
	} else {
		// Embedded, so no decoration is needed.
		layoutResource = R.layout.screen_simple;
		// System.out.println("Simple!");
	}

	mDecor.startChanging();

	// inflate对应的布局文件，并添加到mDecor中。
	View in = mLayoutInflater.inflate(layoutResource, null);
	decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT));
	mContentRoot = (ViewGroup) in;

	// 找到布局中android:id="@android:id/content"。的ViewGroup赋值给contentParent,一般是FrameLayout
	ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT);
	if (contentParent == null) {
		throw new RuntimeException("Window couldn't find content container view");
	}

	if ((features & (1 << FEATURE_INDETERMINATE_PROGRESS)) != 0) {
		ProgressBar progress = getCircularProgressBar(false);
		if (progress != null) {
			progress.setIndeterminate(true);
		}
	}

	if ((features & (1 << FEATURE_SWIPE_TO_DISMISS)) != 0) {
		registerSwipeCallbacks();
	}

	// Remaining setup -- of background and title -- that only applies
	// to top-level windows.
	if (getContainer() == null) {
		final Drawable background;
		if (mBackgroundResource != 0) {
			background = getContext().getDrawable(mBackgroundResource);
		} else {
			background = mBackgroundDrawable;
		}
		mDecor.setWindowBackground(background);

		final Drawable frame;
		if (mFrameResource != 0) {
			frame = getContext().getDrawable(mFrameResource);
		} else {
			frame = null;
		}
		mDecor.setWindowFrame(frame);

		mDecor.setElevation(mElevation);
		mDecor.setClipToOutline(mClipToOutline);

		if (mTitle != null) {
			setTitle(mTitle);
		}

		if (mTitleColor == 0) {
			mTitleColor = mTextColor;
		}
		setTitleColor(mTitleColor);
	}

	mDecor.finishChanging();

	return contentParent;
}	
```
上面看到有很多相应主题的布局文件，我们就以典型的`R.layout.screen_title`为例看一下。
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:fitsSystemWindows="true">
    <!-- Popout bar for action modes -->
    <ViewStub android:id="@+id/action_mode_bar_stub"
              android:inflatedId="@+id/action_mode_bar"
              android:layout="@layout/action_mode_bar"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:theme="?attr/actionBarTheme" />
	// 标题
    <FrameLayout
        android:layout_width="match_parent" 
        android:layout_height="?android:attr/windowTitleSize"
        style="?android:attr/windowTitleBackgroundStyle">
        <TextView android:id="@android:id/title" 
            style="?android:attr/windowTitleStyle"
            android:background="@null"
            android:fadingEdge="horizontal"
            android:gravity="center_vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </FrameLayout>
	// 页面内容
    <FrameLayout android:id="@android:id/content"
        android:layout_width="match_parent" 
        android:layout_height="0dip"
        android:layout_weight="1"
        android:foregroundGravity="fill_horizontal|top"
        android:foreground="?android:attr/windowContentOverlay" />
</LinearLayout>
```
看到这里基本差不多了，我们就以第一次调用`setContentView`方法为例总结一下整体的流程。
- 第一次调用`setContentView()`方法时会去创建一个`DecorView`，这就是整个窗口的根布局。
- 接着回去根据我们设置的对应主题，来加载与之对应的布局文件并将其添加到`DecorView`中，然后找到该布局中`id=content`的`ViewGroup`赋值给`mContentParent`。
- 将我们要设置的`Activity`对应的布局添加到`mContentParent`中。

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 