VideoView源码分析
===

VideoView
---

基于Android4.4源码进行分析

- 简介     
    ```java 
	/**
	 * Displays a video file.  The VideoView class
	 * can load images from various sources (such as resources or content
	 * providers), takes care of computing its measurement from the video so that
	 * it can be used in any layout manager, and provides various display options
	 * such as scaling and tinting.<p>
	 *
	 * <em>Note: VideoView does not retain its full state when going into the
	 * background.</em>  In particular, it does not restore the current play state,
	 * play position, selected tracks, or any subtitle tracks added via
	 * {@link #addSubtitleSource addSubtitleSource()}.  Applications should
	 * save and restore these on their own in
	 * {@link android.app.Activity#onSaveInstanceState} and
	 * {@link android.app.Activity#onRestoreInstanceState}.<p>
	 * Also note that the audio session id (from {@link #getAudioSessionId}) may
	 * change from its previously returned value when the VideoView is restored.
	 */
	```

- 关系       
	```java
	public class VideoView extends SurfaceView
			implements MediaPlayerControl
	```

- 成员
    - 播放器所有的状态
		```java
		// all possible internal states
		private static final int STATE_ERROR              = -1;
		private static final int STATE_IDLE               = 0;
		private static final int STATE_PREPARING          = 1;
		private static final int STATE_PREPARED           = 2;
		private static final int STATE_PLAYING            = 3;
		private static final int STATE_PAUSED             = 4;
		private static final int STATE_PLAYBACK_COMPLETED = 5;
		```

	- 记录播放器状态
		```java
		// mCurrentState is a VideoView object's current state.
		// mTargetState is the state that a method caller intends to reach.
		// For instance, regardless the VideoView object's current state,
		// calling pause() intends to bring the object to a target state
		// of STATE_PAUSED.
		private int mCurrentState = STATE_IDLE;
		private int mTargetState  = STATE_IDLE;
		```		

	- 主要功能部分
		```java
		private SurfaceHolder mSurfaceHolder = null;// 显示图像
        private MediaPlayer mMediaPlayer = null; // 声音、播放
		private MediaController mMediaController; // 播放控制
		```

	- 其他
	    ```java
		private int         mVideoWidth;  // 视频宽度 在onVideoSizeChanged() 和 onPrepared() 中可以得到具体大小
		private int         mVideoHeight;  //视频高度
		private int         mSurfaceWidth; // Surface宽度  在SurfaceHolder.Callback.surfaceChanged() 中可以得到具体大小
		private int         mSurfaceHeight; // Surface高度
		private int         mSeekWhenPrepared;  // recording the seek position while preparing
		```
		
- 具体实现
    - 构造方法
	    ```java
		public VideoView(Context context) {
			super(context);
			initVideoView();
		}

		public VideoView(Context context, AttributeSet attrs) {
			this(context, attrs, 0);
			initVideoView();
		}

		public VideoView(Context context, AttributeSet attrs, int defStyle) {
			super(context, attrs, defStyle);
			initVideoView();
		}
		```
		
		```java
		// 进行一些必要信息的初始化设置
		private void initVideoView() {
			mVideoWidth = 0;
			mVideoHeight = 0;
			
			// 通过SurfaceHolder去控制SurfaceView
			getHolder().addCallback(mSHCallback);
			// Deprecated. this is ignored, this value is set automatically when needed.Android3.0以上会自动设置，但是为了兼容还需设置
			getHolder().setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
			
			setFocusable(true);
			setFocusableInTouchMode(true);
			requestFocus();
			
			// 字幕相关，用不到
			mPendingSubtitleTracks = new Vector<Pair<InputStream, MediaFormat>>();
			
			mCurrentState = STATE_IDLE;
			mTargetState  = STATE_IDLE;
		}
		```
		
		SurfaceHolder.Callback源码
		```java
		SurfaceHolder.Callback mSHCallback = new SurfaceHolder.Callback()
		{
			public void surfaceChanged(SurfaceHolder holder, int format,
										int w, int h)
			{
				mSurfaceWidth = w;
				mSurfaceHeight = h;
				boolean isValidState =  (mTargetState == STATE_PLAYING);
				boolean hasValidSize = (mVideoWidth == w && mVideoHeight == h);
				if (mMediaPlayer != null && isValidState && hasValidSize) {
					if (mSeekWhenPrepared != 0) {
						seekTo(mSeekWhenPrepared);
					}
					// 如果当前已经是播放状态的话就调用mediaplaer.start() 方法，并且把当前状态以及目标状态进行改变
					start();
				}
			}

			public void surfaceCreated(SurfaceHolder holder)
			{
				mSurfaceHolder = holder;
				// Surface创建后就开始调用播放
				openVideo();
			}

			public void surfaceDestroyed(SurfaceHolder holder)
			{
				// after we return from this we can't use the surface any more
				mSurfaceHolder = null;
				if (mMediaController != null) mMediaController.hide();
				release(true);
			}
		};
		```
		
	- 重写onMeasure()方法
		```java
		@Override
	    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	        int width = getDefaultSize(mVideoWidth, widthMeasureSpec);
	        int height = getDefaultSize(mVideoHeight, heightMeasureSpec);
			
			// ....根据视频的宽高比进行处理， 为了更好的宽展，提供一些用户能自己选择的模式，一般会另外提供方法, 这部分代码可以先不看 start
			int width = getDefaultSize(mVideoWidth, widthMeasureSpec);
	        int height = getDefaultSize(mVideoHeight, heightMeasureSpec);
	        if (mVideoWidth > 0 && mVideoHeight > 0) {
	
	            int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
	            int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
	            int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
	            int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);
	
	            if (widthSpecMode == MeasureSpec.EXACTLY && heightSpecMode == MeasureSpec.EXACTLY) {
	                // the size is fixed
	                width = widthSpecSize;
	                height = heightSpecSize;
	
	                // for compatibility, we adjust size based on aspect ratio
	                if ( mVideoWidth * height  < width * mVideoHeight ) {
	                    //Log.i("@@@", "image too wide, correcting");
	                    width = height * mVideoWidth / mVideoHeight;
	                } else if ( mVideoWidth * height  > width * mVideoHeight ) {
	                    //Log.i("@@@", "image too tall, correcting");
	                    height = width * mVideoHeight / mVideoWidth;
	                }
	            } else if (widthSpecMode == MeasureSpec.EXACTLY) {
	                // only the width is fixed, adjust the height to match aspect ratio if possible
	                width = widthSpecSize;
	                height = width * mVideoHeight / mVideoWidth;
	                if (heightSpecMode == MeasureSpec.AT_MOST && height > heightSpecSize) {
	                    // couldn't match aspect ratio within the constraints
	                    height = heightSpecSize;
	                }
	            } else if (heightSpecMode == MeasureSpec.EXACTLY) {
	                // only the height is fixed, adjust the width to match aspect ratio if possible
	                height = heightSpecSize;
	                width = height * mVideoWidth / mVideoHeight;
	                if (widthSpecMode == MeasureSpec.AT_MOST && width > widthSpecSize) {
	                    // couldn't match aspect ratio within the constraints
	                    width = widthSpecSize;
	                }
	            } else {
	                // neither the width nor the height are fixed, try to use actual video size
	                width = mVideoWidth;
	                height = mVideoHeight;
	                if (heightSpecMode == MeasureSpec.AT_MOST && height > heightSpecSize) {
	                    // too tall, decrease both width and height
	                    height = heightSpecSize;
	                    width = height * mVideoWidth / mVideoHeight;
	                }
	                if (widthSpecMode == MeasureSpec.AT_MOST && width > widthSpecSize) {
	                    // too wide, decrease both width and height
	                    width = widthSpecSize;
	                    height = width * mVideoHeight / mVideoWidth;
	                }
	            }
	        } else {
	            // no size yet, just adopt the given spec sizes
	        }
			// end
			
	        setMeasuredDimension(width, height);
	    }
		```
	
		- 附上getDefaultSize()源码 
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
 
	- 外部进行播放调用
		```java
		public void setVideoPath(String path) {
			setVideoURI(Uri.parse(path));
		}

		public void setVideoURI(Uri uri) {
			setVideoURI(uri, null);
		}

		/**
		 * @hide
		 */
		public void setVideoURI(Uri uri, Map<String, String> headers) {
			mUri = uri;
			mHeaders = headers;
			mSeekWhenPrepared = 0;
			
			openVideo();
			
			requestLayout();
			invalidate();
		}
		```
		
		- openVide() 源码
		    ```java
			private void openVideo() {
				if (mUri == null || mSurfaceHolder == null) {
					// not ready for playback just yet, will try again later
					return;
				}
				
				// Tell the music playback service to pause
				// TODO: these constants need to be published somewhere in the framework.
				Intent i = new Intent("com.android.music.musicservicecommand");
				i.putExtra("command", "pause");
				mContext.sendBroadcast(i);

				// we shouldn't clear the target state, because somebody might have
				// called start() previously // 先把已经存在的MediaPlayer释放掉，然后重新创建一个, 不一定只在SetVideoPath() 中调用，在其他地方也会调用
				release(false);
				
				try {
					// 创建一个MediaPlayer
					mMediaPlayer = new MediaPlayer();
					// TODO: create SubtitleController in MediaPlayer, but we need
					// a context for the subtitle renderers
					final Context context = getContext();
					final SubtitleController controller = new SubtitleController(
							context, mMediaPlayer.getMediaTimeProvider(), mMediaPlayer);
					controller.registerRenderer(new WebVttRenderer(context));
					mMediaPlayer.setSubtitleAnchor(controller, this);

					if (mAudioSession != 0) {
						mMediaPlayer.setAudioSessionId(mAudioSession);
					} else {
						mAudioSession = mMediaPlayer.getAudioSessionId();
					}
					
					// 设置一些必要的监听
					mMediaPlayer.setOnPreparedListener(mPreparedListener);
					mMediaPlayer.setOnVideoSizeChangedListener(mSizeChangedListener);
					mMediaPlayer.setOnCompletionListener(mCompletionListener);
					mMediaPlayer.setOnErrorListener(mErrorListener);
					mMediaPlayer.setOnInfoListener(mInfoListener);
					mMediaPlayer.setOnBufferingUpdateListener(mBufferingUpdateListener);
					mCurrentBufferPercentage = 0;
					// 让MediaPlayer进行播放
					mMediaPlayer.setDataSource(mContext, mUri, mHeaders);
					// 让SurfaceView进行画面显示
					mMediaPlayer.setDisplay(mSurfaceHolder);
					// 设置音频类型
					mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
					// 播放时屏幕常亮
					mMediaPlayer.setScreenOnWhilePlaying(true);
					// Prepares the player for playback, asynchronously. After setting the datasource and the display surface, you need to either call prepare() or prepareAsync(). For streams, you should call prepareAsync(), 
					// which returns immediately, rather than blocking until enough data has been buffered.
					mMediaPlayer.prepareAsync();

					for (Pair<InputStream, MediaFormat> pending: mPendingSubtitleTracks) {
						try {
							mMediaPlayer.addSubtitleSource(pending.first, pending.second);
						} catch (IllegalStateException e) {
							mInfoListener.onInfo(
									mMediaPlayer, MediaPlayer.MEDIA_INFO_UNSUPPORTED_SUBTITLE, 0);
						}
					}

					// we don't set the target state here either, but preserve the
					// target state that was there before.
					mCurrentState = STATE_PREPARING;
					// 如果已经调用过SetMediaController() 方法，这里会直接显示
					attachMediaController();
				} catch (IOException ex) {
					Log.w(TAG, "Unable to open content: " + mUri, ex);
					mCurrentState = STATE_ERROR;
					mTargetState = STATE_ERROR;
					mErrorListener.onError(mMediaPlayer, MediaPlayer.MEDIA_ERROR_UNKNOWN, 0);
					return;
				} catch (IllegalArgumentException ex) {
					Log.w(TAG, "Unable to open content: " + mUri, ex);
					mCurrentState = STATE_ERROR;
					mTargetState = STATE_ERROR;
					mErrorListener.onError(mMediaPlayer, MediaPlayer.MEDIA_ERROR_UNKNOWN, 0);
					return;
				} finally {
					mPendingSubtitleTracks.clear();
				}
			}
			```
			
			release() 方法,在开始播放一个视频的时候会先调用该方法，然后重新创建一个，在SurfaceView销毁的时候也会调用该方法
			```java
			/*
			 * release the media player in any state
			 */
			private void release(boolean cleartargetstate) {
				if (mMediaPlayer != null) {
					mMediaPlayer.reset();
					mMediaPlayer.release();
					mMediaPlayer = null;
					mPendingSubtitleTracks.clear();
					mCurrentState = STATE_IDLE;
					if (cleartargetstate) {
						mTargetState  = STATE_IDLE;
					}
				}
			}
			```
		
		- 外部停止播放调用
			```java
			public void stopPlayback() {
				if (mMediaPlayer != null) {
					mMediaPlayer.stop();
					mMediaPlayer.release();
					mMediaPlayer = null;
					mCurrentState = STATE_IDLE;
					mTargetState  = STATE_IDLE;
				}
			}
			```
			
		- 外部设置控制栏部分
		    ```java
			public void setMediaController(MediaController controller) {
				if (mMediaController != null) {
					mMediaController.hide();
				}
				mMediaController = controller;
				attachMediaController();
			}

			private void attachMediaController() {
				if (mMediaPlayer != null && mMediaController != null) {
					// setMediaPlayer(MediaPlayerControl player), 让MediaPlayer相应的控制部分调用本类中的实现方法
					mMediaController.setMediaPlayer(this);
					View anchorView = this.getParent() instanceof View ?
							(View)this.getParent() : this;
							
					// 创建Controller并且依据AnchorView的位置进行显示
					mMediaController.setAnchorView(anchorView);
					mMediaController.setEnabled(isInPlaybackState());
				}
			}
			```
			
		- MediaPlayer必要监听
			- OnVideoSizeChangedListener
				```java
				MediaPlayer.OnVideoSizeChangedListener mSizeChangedListener =
					new MediaPlayer.OnVideoSizeChangedListener() {
						public void onVideoSizeChanged(MediaPlayer mp, int width, int height) {
							mVideoWidth = mp.getVideoWidth();
							mVideoHeight = mp.getVideoHeight();
							if (mVideoWidth != 0 && mVideoHeight != 0) {
								// 这个方法是设置Surface分辨率，而不是设置视频播放窗口的大小，视频播放窗口大小是由SurfaceView的布局控制，要分清Surface与SurfaceView的区别，Surface是Window中整个的一个控件(句柄),
								// 而SurfaceView是一个包含Surface的View，SurfaceView覆盖到Surface上(可以这样理解)，我们只能通过SurfaceView来看Surface中的内容,至于在SurfaceView显示之外的Surface我们是不可见的.
								getHolder().setFixedSize(mVideoWidth, mVideoHeight);
								requestLayout();
							}
						}
				};
				```

		    - OnPreparedListener
			    ```java
				MediaPlayer.OnPreparedListener mPreparedListener = new MediaPlayer.OnPreparedListener() {
					public void onPrepared(MediaPlayer mp) {
						mCurrentState = STATE_PREPARED;

						// Get the capabilities of the player for this stream
						Metadata data = mp.getMetadata(MediaPlayer.METADATA_ALL,
												  MediaPlayer.BYPASS_METADATA_FILTER);

						if (data != null) {
							mCanPause = !data.has(Metadata.PAUSE_AVAILABLE)
									|| data.getBoolean(Metadata.PAUSE_AVAILABLE);
							mCanSeekBack = !data.has(Metadata.SEEK_BACKWARD_AVAILABLE)
									|| data.getBoolean(Metadata.SEEK_BACKWARD_AVAILABLE);
							mCanSeekForward = !data.has(Metadata.SEEK_FORWARD_AVAILABLE)
									|| data.getBoolean(Metadata.SEEK_FORWARD_AVAILABLE);
						} else {
							mCanPause = mCanSeekBack = mCanSeekForward = true;
						}

						if (mOnPreparedListener != null) {
							mOnPreparedListener.onPrepared(mMediaPlayer);
						}
						if (mMediaController != null) {
							mMediaController.setEnabled(true);
						}
						mVideoWidth = mp.getVideoWidth();
						mVideoHeight = mp.getVideoHeight();

						int seekToPosition = mSeekWhenPrepared;  // mSeekWhenPrepared may be changed after seekTo() call
						if (seekToPosition != 0) {
							seekTo(seekToPosition);
						}
						if (mVideoWidth != 0 && mVideoHeight != 0) {
							//Log.i("@@@@", "video size: " + mVideoWidth +"/"+ mVideoHeight);
							getHolder().setFixedSize(mVideoWidth, mVideoHeight);
							if (mSurfaceWidth == mVideoWidth && mSurfaceHeight == mVideoHeight) {
								// We didn't actually change the size (it was already at the size
								// we need), so we won't get a "surface changed" callback, so
								// start the video here instead of in the callback.
								if (mTargetState == STATE_PLAYING) {
									start();
									if (mMediaController != null) {
										mMediaController.show();
									}
								} else if (!isPlaying() &&
										   (seekToPosition != 0 || getCurrentPosition() > 0)) {
								   if (mMediaController != null) {
									   // Show the media controls when we're paused into a video and make 'em stick.
									   mMediaController.show(0);
								   }
							   }
							}
						} else {
							// We don't know the video size yet, but should start anyway.
							// The video size might be reported to us later.
							if (mTargetState == STATE_PLAYING) {
								start();
							}
						}
					}
				};
				```
				
			- OnCompletionListener
				```java
				private MediaPlayer.OnCompletionListener mCompletionListener =
					new MediaPlayer.OnCompletionListener() {
					public void onCompletion(MediaPlayer mp) {
						mCurrentState = STATE_PLAYBACK_COMPLETED;
						mTargetState = STATE_PLAYBACK_COMPLETED;
						if (mMediaController != null) {
							mMediaController.hide();
						}
						if (mOnCompletionListener != null) {
							mOnCompletionListener.onCompletion(mMediaPlayer);
						}
					}
				};
				```
				
		- Touch以及Key的监听
		    ```java
			@Override
			public boolean onTouchEvent(MotionEvent ev) {
				if (isInPlaybackState() && mMediaController != null) {
				    // 控制MediaController的显示与隐藏
					toggleMediaControlsVisiblity();
				}
				return false;
			}
			```

		- toggleMediaControlsVisiblity      
			```java
			private void toggleMediaControlsVisiblity() {
				if (mMediaController.isShowing()) {
					mMediaController.hide();
				} else {
					mMediaController.show();
				}
			}
			```
				
		- Key
		    ```java
			@Override
			public boolean onKeyDown(int keyCode, KeyEvent event)
			{
				boolean isKeyCodeSupported = keyCode != KeyEvent.KEYCODE_BACK &&
											 keyCode != KeyEvent.KEYCODE_VOLUME_UP &&
											 keyCode != KeyEvent.KEYCODE_VOLUME_DOWN &&
											 keyCode != KeyEvent.KEYCODE_VOLUME_MUTE &&
											 keyCode != KeyEvent.KEYCODE_MENU &&
											 keyCode != KeyEvent.KEYCODE_CALL &&
											 keyCode != KeyEvent.KEYCODE_ENDCALL;
				if (isInPlaybackState() && isKeyCodeSupported && mMediaController != null) {
					if (keyCode == KeyEvent.KEYCODE_HEADSETHOOK ||
							keyCode == KeyEvent.KEYCODE_MEDIA_PLAY_PAUSE) {
						if (mMediaPlayer.isPlaying()) {
							pause();
							mMediaController.show();
						} else {
							start();
							mMediaController.hide();
						}
						return true;
					} else if (keyCode == KeyEvent.KEYCODE_MEDIA_PLAY) {
						if (!mMediaPlayer.isPlaying()) {
							start();
							mMediaController.hide();
						}
						return true;
					} else if (keyCode == KeyEvent.KEYCODE_MEDIA_STOP
							|| keyCode == KeyEvent.KEYCODE_MEDIA_PAUSE) {
						if (mMediaPlayer.isPlaying()) {
							pause();
							mMediaController.show();
						}
						return true;
					} else {
						toggleMediaControlsVisiblity();
					}
				}

				return super.onKeyDown(keyCode, event);
			}
			```

华丽丽的分割线 上源码
==============

----------------------
```java
/*
 * Copyright (C) 2006 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

/**
 * Displays a video file.  The VideoView class
 * can load images from various sources (such as resources or content
 * providers), takes care of computing its measurement from the video so that
 * it can be used in any layout manager, and provides various display options
 * such as scaling and tinting.<p>
 *
 * <em>Note: VideoView does not retain its full state when going into the
 * background.</em>  In particular, it does not restore the current play state,
 * play position, selected tracks, or any subtitle tracks added via
 * {@link #addSubtitleSource addSubtitleSource()}.  Applications should
 * save and restore these on their own in
 * {@link android.app.Activity#onSaveInstanceState} and
 * {@link android.app.Activity#onRestoreInstanceState}.<p>
 * Also note that the audio session id (from {@link #getAudioSessionId}) may
 * change from its previously returned value when the VideoView is restored.
 */
public class VideoView extends SurfaceView
        implements MediaPlayerControl, SubtitleController.Anchor {
    private String TAG = "VideoView";
    // settable by the client
    private Uri         mUri;
    private Map<String, String> mHeaders;

    // all possible internal states
    private static final int STATE_ERROR              = -1;
    private static final int STATE_IDLE               = 0;
    private static final int STATE_PREPARING          = 1;
    private static final int STATE_PREPARED           = 2;
    private static final int STATE_PLAYING            = 3;
    private static final int STATE_PAUSED             = 4;
    private static final int STATE_PLAYBACK_COMPLETED = 5;

    // mCurrentState is a VideoView object's current state.
    // mTargetState is the state that a method caller intends to reach.
    // For instance, regardless the VideoView object's current state,
    // calling pause() intends to bring the object to a target state
    // of STATE_PAUSED.
    private int mCurrentState = STATE_IDLE;
    private int mTargetState  = STATE_IDLE;

    // All the stuff we need for playing and showing a video
    private SurfaceHolder mSurfaceHolder = null;
    private MediaPlayer mMediaPlayer = null;
    private int         mAudioSession;
    private int         mVideoWidth;
    private int         mVideoHeight;
    private int         mSurfaceWidth;
    private int         mSurfaceHeight;
    private MediaController mMediaController;
    private OnCompletionListener mOnCompletionListener;
    private MediaPlayer.OnPreparedListener mOnPreparedListener;
    private int         mCurrentBufferPercentage;
    private OnErrorListener mOnErrorListener;
    private OnInfoListener  mOnInfoListener;
    private int         mSeekWhenPrepared;  // recording the seek position while preparing
    private boolean     mCanPause;
    private boolean     mCanSeekBack;
    private boolean     mCanSeekForward;

    /** Subtitle rendering widget overlaid on top of the video. */
    private RenderingWidget mSubtitleWidget;

    /** Listener for changes to subtitle data, used to redraw when needed. */
    private RenderingWidget.OnChangedListener mSubtitlesChangedListener;

    public VideoView(Context context) {
        super(context);
        initVideoView();
    }

    public VideoView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
        initVideoView();
    }

    public VideoView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        initVideoView();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //Log.i("@@@@", "onMeasure(" + MeasureSpec.toString(widthMeasureSpec) + ", "
        //        + MeasureSpec.toString(heightMeasureSpec) + ")");

        int width = getDefaultSize(mVideoWidth, widthMeasureSpec);
        int height = getDefaultSize(mVideoHeight, heightMeasureSpec);
        if (mVideoWidth > 0 && mVideoHeight > 0) {

            int widthSpecMode = MeasureSpec.getMode(widthMeasureSpec);
            int widthSpecSize = MeasureSpec.getSize(widthMeasureSpec);
            int heightSpecMode = MeasureSpec.getMode(heightMeasureSpec);
            int heightSpecSize = MeasureSpec.getSize(heightMeasureSpec);

            if (widthSpecMode == MeasureSpec.EXACTLY && heightSpecMode == MeasureSpec.EXACTLY) {
                // the size is fixed
                width = widthSpecSize;
                height = heightSpecSize;

                // for compatibility, we adjust size based on aspect ratio
                if ( mVideoWidth * height  < width * mVideoHeight ) {
                    //Log.i("@@@", "image too wide, correcting");
                    width = height * mVideoWidth / mVideoHeight;
                } else if ( mVideoWidth * height  > width * mVideoHeight ) {
                    //Log.i("@@@", "image too tall, correcting");
                    height = width * mVideoHeight / mVideoWidth;
                }
            } else if (widthSpecMode == MeasureSpec.EXACTLY) {
                // only the width is fixed, adjust the height to match aspect ratio if possible
                width = widthSpecSize;
                height = width * mVideoHeight / mVideoWidth;
                if (heightSpecMode == MeasureSpec.AT_MOST && height > heightSpecSize) {
                    // couldn't match aspect ratio within the constraints
                    height = heightSpecSize;
                }
            } else if (heightSpecMode == MeasureSpec.EXACTLY) {
                // only the height is fixed, adjust the width to match aspect ratio if possible
                height = heightSpecSize;
                width = height * mVideoWidth / mVideoHeight;
                if (widthSpecMode == MeasureSpec.AT_MOST && width > widthSpecSize) {
                    // couldn't match aspect ratio within the constraints
                    width = widthSpecSize;
                }
            } else {
                // neither the width nor the height are fixed, try to use actual video size
                width = mVideoWidth;
                height = mVideoHeight;
                if (heightSpecMode == MeasureSpec.AT_MOST && height > heightSpecSize) {
                    // too tall, decrease both width and height
                    height = heightSpecSize;
                    width = height * mVideoWidth / mVideoHeight;
                }
                if (widthSpecMode == MeasureSpec.AT_MOST && width > widthSpecSize) {
                    // too wide, decrease both width and height
                    width = widthSpecSize;
                    height = width * mVideoHeight / mVideoWidth;
                }
            }
        } else {
            // no size yet, just adopt the given spec sizes
        }
        setMeasuredDimension(width, height);
    }

    @Override
    public void onInitializeAccessibilityEvent(AccessibilityEvent event) {
        super.onInitializeAccessibilityEvent(event);
        event.setClassName(VideoView.class.getName());
    }

    @Override
    public void onInitializeAccessibilityNodeInfo(AccessibilityNodeInfo info) {
        super.onInitializeAccessibilityNodeInfo(info);
        info.setClassName(VideoView.class.getName());
    }

    public int resolveAdjustedSize(int desiredSize, int measureSpec) {
        return getDefaultSize(desiredSize, measureSpec);
    }

    private void initVideoView() {
        mVideoWidth = 0;
        mVideoHeight = 0;
        getHolder().addCallback(mSHCallback);
        getHolder().setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
        setFocusable(true);
        setFocusableInTouchMode(true);
        requestFocus();
        mPendingSubtitleTracks = new Vector<Pair<InputStream, MediaFormat>>();
        mCurrentState = STATE_IDLE;
        mTargetState  = STATE_IDLE;
    }

    public void setVideoPath(String path) {
        setVideoURI(Uri.parse(path));
    }

    public void setVideoURI(Uri uri) {
        setVideoURI(uri, null);
    }

    /**
     * @hide
     */
    public void setVideoURI(Uri uri, Map<String, String> headers) {
        mUri = uri;
        mHeaders = headers;
        mSeekWhenPrepared = 0;
        openVideo();
        requestLayout();
        invalidate();
    }

    /**
     * Adds an external subtitle source file (from the provided input stream.)
     *
     * Note that a single external subtitle source may contain multiple or no
     * supported tracks in it. If the source contained at least one track in
     * it, one will receive an {@link MediaPlayer#MEDIA_INFO_METADATA_UPDATE}
     * info message. Otherwise, if reading the source takes excessive time,
     * one will receive a {@link MediaPlayer#MEDIA_INFO_SUBTITLE_TIMED_OUT}
     * message. If the source contained no supported track (including an empty
     * source file or null input stream), one will receive a {@link
     * MediaPlayer#MEDIA_INFO_UNSUPPORTED_SUBTITLE} message. One can find the
     * total number of available tracks using {@link MediaPlayer#getTrackInfo()}
     * to see what additional tracks become available after this method call.
     *
     * @param is     input stream containing the subtitle data.  It will be
     *               closed by the media framework.
     * @param format the format of the subtitle track(s).  Must contain at least
     *               the mime type ({@link MediaFormat#KEY_MIME}) and the
     *               language ({@link MediaFormat#KEY_LANGUAGE}) of the file.
     *               If the file itself contains the language information,
     *               specify "und" for the language.
     */
    public void addSubtitleSource(InputStream is, MediaFormat format) {
        if (mMediaPlayer == null) {
            mPendingSubtitleTracks.add(Pair.create(is, format));
        } else {
            try {
                mMediaPlayer.addSubtitleSource(is, format);
            } catch (IllegalStateException e) {
                mInfoListener.onInfo(
                        mMediaPlayer, MediaPlayer.MEDIA_INFO_UNSUPPORTED_SUBTITLE, 0);
            }
        }
    }

    private Vector<Pair<InputStream, MediaFormat>> mPendingSubtitleTracks;

    public void stopPlayback() {
        if (mMediaPlayer != null) {
            mMediaPlayer.stop();
            mMediaPlayer.release();
            mMediaPlayer = null;
            mCurrentState = STATE_IDLE;
            mTargetState  = STATE_IDLE;
        }
    }

    private void openVideo() {
        if (mUri == null || mSurfaceHolder == null) {
            // not ready for playback just yet, will try again later
            return;
        }
        // Tell the music playback service to pause
        // TODO: these constants need to be published somewhere in the framework.
        Intent i = new Intent("com.android.music.musicservicecommand");
        i.putExtra("command", "pause");
        mContext.sendBroadcast(i);

        // we shouldn't clear the target state, because somebody might have
        // called start() previously
        release(false);
        try {
            mMediaPlayer = new MediaPlayer();
            // TODO: create SubtitleController in MediaPlayer, but we need
            // a context for the subtitle renderers
            final Context context = getContext();
            final SubtitleController controller = new SubtitleController(
                    context, mMediaPlayer.getMediaTimeProvider(), mMediaPlayer);
            controller.registerRenderer(new WebVttRenderer(context));
            mMediaPlayer.setSubtitleAnchor(controller, this);

            if (mAudioSession != 0) {
                mMediaPlayer.setAudioSessionId(mAudioSession);
            } else {
                mAudioSession = mMediaPlayer.getAudioSessionId();
            }
            mMediaPlayer.setOnPreparedListener(mPreparedListener);
            mMediaPlayer.setOnVideoSizeChangedListener(mSizeChangedListener);
            mMediaPlayer.setOnCompletionListener(mCompletionListener);
            mMediaPlayer.setOnErrorListener(mErrorListener);
            mMediaPlayer.setOnInfoListener(mInfoListener);
            mMediaPlayer.setOnBufferingUpdateListener(mBufferingUpdateListener);
            mCurrentBufferPercentage = 0;
            mMediaPlayer.setDataSource(mContext, mUri, mHeaders);
            mMediaPlayer.setDisplay(mSurfaceHolder);
            mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
            mMediaPlayer.setScreenOnWhilePlaying(true);
            mMediaPlayer.prepareAsync();

            for (Pair<InputStream, MediaFormat> pending: mPendingSubtitleTracks) {
                try {
                    mMediaPlayer.addSubtitleSource(pending.first, pending.second);
                } catch (IllegalStateException e) {
                    mInfoListener.onInfo(
                            mMediaPlayer, MediaPlayer.MEDIA_INFO_UNSUPPORTED_SUBTITLE, 0);
                }
            }

            // we don't set the target state here either, but preserve the
            // target state that was there before.
            mCurrentState = STATE_PREPARING;
            attachMediaController();
        } catch (IOException ex) {
            Log.w(TAG, "Unable to open content: " + mUri, ex);
            mCurrentState = STATE_ERROR;
            mTargetState = STATE_ERROR;
            mErrorListener.onError(mMediaPlayer, MediaPlayer.MEDIA_ERROR_UNKNOWN, 0);
            return;
        } catch (IllegalArgumentException ex) {
            Log.w(TAG, "Unable to open content: " + mUri, ex);
            mCurrentState = STATE_ERROR;
            mTargetState = STATE_ERROR;
            mErrorListener.onError(mMediaPlayer, MediaPlayer.MEDIA_ERROR_UNKNOWN, 0);
            return;
        } finally {
            mPendingSubtitleTracks.clear();
        }
    }

    public void setMediaController(MediaController controller) {
        if (mMediaController != null) {
            mMediaController.hide();
        }
        mMediaController = controller;
        attachMediaController();
    }

    private void attachMediaController() {
        if (mMediaPlayer != null && mMediaController != null) {
            mMediaController.setMediaPlayer(this);
            View anchorView = this.getParent() instanceof View ?
                    (View)this.getParent() : this;
            mMediaController.setAnchorView(anchorView);
            mMediaController.setEnabled(isInPlaybackState());
        }
    }

    MediaPlayer.OnVideoSizeChangedListener mSizeChangedListener =
        new MediaPlayer.OnVideoSizeChangedListener() {
            public void onVideoSizeChanged(MediaPlayer mp, int width, int height) {
                mVideoWidth = mp.getVideoWidth();
                mVideoHeight = mp.getVideoHeight();
                if (mVideoWidth != 0 && mVideoHeight != 0) {
                    getHolder().setFixedSize(mVideoWidth, mVideoHeight);
                    requestLayout();
                }
            }
    };

    MediaPlayer.OnPreparedListener mPreparedListener = new MediaPlayer.OnPreparedListener() {
        public void onPrepared(MediaPlayer mp) {
            mCurrentState = STATE_PREPARED;

            // Get the capabilities of the player for this stream
            Metadata data = mp.getMetadata(MediaPlayer.METADATA_ALL,
                                      MediaPlayer.BYPASS_METADATA_FILTER);

            if (data != null) {
                mCanPause = !data.has(Metadata.PAUSE_AVAILABLE)
                        || data.getBoolean(Metadata.PAUSE_AVAILABLE);
                mCanSeekBack = !data.has(Metadata.SEEK_BACKWARD_AVAILABLE)
                        || data.getBoolean(Metadata.SEEK_BACKWARD_AVAILABLE);
                mCanSeekForward = !data.has(Metadata.SEEK_FORWARD_AVAILABLE)
                        || data.getBoolean(Metadata.SEEK_FORWARD_AVAILABLE);
            } else {
                mCanPause = mCanSeekBack = mCanSeekForward = true;
            }

            if (mOnPreparedListener != null) {
                mOnPreparedListener.onPrepared(mMediaPlayer);
            }
            if (mMediaController != null) {
                mMediaController.setEnabled(true);
            }
            mVideoWidth = mp.getVideoWidth();
            mVideoHeight = mp.getVideoHeight();

            int seekToPosition = mSeekWhenPrepared;  // mSeekWhenPrepared may be changed after seekTo() call
            if (seekToPosition != 0) {
                seekTo(seekToPosition);
            }
            if (mVideoWidth != 0 && mVideoHeight != 0) {
                //Log.i("@@@@", "video size: " + mVideoWidth +"/"+ mVideoHeight);
                getHolder().setFixedSize(mVideoWidth, mVideoHeight);
                if (mSurfaceWidth == mVideoWidth && mSurfaceHeight == mVideoHeight) {
                    // We didn't actually change the size (it was already at the size
                    // we need), so we won't get a "surface changed" callback, so
                    // start the video here instead of in the callback.
                    if (mTargetState == STATE_PLAYING) {
                        start();
                        if (mMediaController != null) {
                            mMediaController.show();
                        }
                    } else if (!isPlaying() &&
                               (seekToPosition != 0 || getCurrentPosition() > 0)) {
                       if (mMediaController != null) {
                           // Show the media controls when we're paused into a video and make 'em stick.
                           mMediaController.show(0);
                       }
                   }
                }
            } else {
                // We don't know the video size yet, but should start anyway.
                // The video size might be reported to us later.
                if (mTargetState == STATE_PLAYING) {
                    start();
                }
            }
        }
    };

    private MediaPlayer.OnCompletionListener mCompletionListener =
        new MediaPlayer.OnCompletionListener() {
        public void onCompletion(MediaPlayer mp) {
            mCurrentState = STATE_PLAYBACK_COMPLETED;
            mTargetState = STATE_PLAYBACK_COMPLETED;
            if (mMediaController != null) {
                mMediaController.hide();
            }
            if (mOnCompletionListener != null) {
                mOnCompletionListener.onCompletion(mMediaPlayer);
            }
        }
    };

    private MediaPlayer.OnInfoListener mInfoListener =
        new MediaPlayer.OnInfoListener() {
        public  boolean onInfo(MediaPlayer mp, int arg1, int arg2) {
            if (mOnInfoListener != null) {
                mOnInfoListener.onInfo(mp, arg1, arg2);
            }
            return true;
        }
    };

    private MediaPlayer.OnErrorListener mErrorListener =
        new MediaPlayer.OnErrorListener() {
        public boolean onError(MediaPlayer mp, int framework_err, int impl_err) {
            Log.d(TAG, "Error: " + framework_err + "," + impl_err);
            mCurrentState = STATE_ERROR;
            mTargetState = STATE_ERROR;
            if (mMediaController != null) {
                mMediaController.hide();
            }

            /* If an error handler has been supplied, use it and finish. */
            if (mOnErrorListener != null) {
                if (mOnErrorListener.onError(mMediaPlayer, framework_err, impl_err)) {
                    return true;
                }
            }

            /* Otherwise, pop up an error dialog so the user knows that
             * something bad has happened. Only try and pop up the dialog
             * if we're attached to a window. When we're going away and no
             * longer have a window, don't bother showing the user an error.
             */
            if (getWindowToken() != null) {
                Resources r = mContext.getResources();
                int messageId;

                if (framework_err == MediaPlayer.MEDIA_ERROR_NOT_VALID_FOR_PROGRESSIVE_PLAYBACK) {
                    messageId = com.android.internal.R.string.VideoView_error_text_invalid_progressive_playback;
                } else {
                    messageId = com.android.internal.R.string.VideoView_error_text_unknown;
                }

                new AlertDialog.Builder(mContext)
                        .setMessage(messageId)
                        .setPositiveButton(com.android.internal.R.string.VideoView_error_button,
                                new DialogInterface.OnClickListener() {
                                    public void onClick(DialogInterface dialog, int whichButton) {
                                        /* If we get here, there is no onError listener, so
                                         * at least inform them that the video is over.
                                         */
                                        if (mOnCompletionListener != null) {
                                            mOnCompletionListener.onCompletion(mMediaPlayer);
                                        }
                                    }
                                })
                        .setCancelable(false)
                        .show();
            }
            return true;
        }
    };

    private MediaPlayer.OnBufferingUpdateListener mBufferingUpdateListener =
        new MediaPlayer.OnBufferingUpdateListener() {
        public void onBufferingUpdate(MediaPlayer mp, int percent) {
            mCurrentBufferPercentage = percent;
        }
    };

    /**
     * Register a callback to be invoked when the media file
     * is loaded and ready to go.
     *
     * @param l The callback that will be run
     */
    public void setOnPreparedListener(MediaPlayer.OnPreparedListener l)
    {
        mOnPreparedListener = l;
    }

    /**
     * Register a callback to be invoked when the end of a media file
     * has been reached during playback.
     *
     * @param l The callback that will be run
     */
    public void setOnCompletionListener(OnCompletionListener l)
    {
        mOnCompletionListener = l;
    }

    /**
     * Register a callback to be invoked when an error occurs
     * during playback or setup.  If no listener is specified,
     * or if the listener returned false, VideoView will inform
     * the user of any errors.
     *
     * @param l The callback that will be run
     */
    public void setOnErrorListener(OnErrorListener l)
    {
        mOnErrorListener = l;
    }

    /**
     * Register a callback to be invoked when an informational event
     * occurs during playback or setup.
     *
     * @param l The callback that will be run
     */
    public void setOnInfoListener(OnInfoListener l) {
        mOnInfoListener = l;
    }

    SurfaceHolder.Callback mSHCallback = new SurfaceHolder.Callback()
    {
        public void surfaceChanged(SurfaceHolder holder, int format,
                                    int w, int h)
        {
            mSurfaceWidth = w;
            mSurfaceHeight = h;
            boolean isValidState =  (mTargetState == STATE_PLAYING);
            boolean hasValidSize = (mVideoWidth == w && mVideoHeight == h);
            if (mMediaPlayer != null && isValidState && hasValidSize) {
                if (mSeekWhenPrepared != 0) {
                    seekTo(mSeekWhenPrepared);
                }
                start();
            }
        }

        public void surfaceCreated(SurfaceHolder holder)
        {
            mSurfaceHolder = holder;
            openVideo();
        }

        public void surfaceDestroyed(SurfaceHolder holder)
        {
            // after we return from this we can't use the surface any more
            mSurfaceHolder = null;
            if (mMediaController != null) mMediaController.hide();
            release(true);
        }
    };

    /*
     * release the media player in any state
     */
    private void release(boolean cleartargetstate) {
        if (mMediaPlayer != null) {
            mMediaPlayer.reset();
            mMediaPlayer.release();
            mMediaPlayer = null;
            mPendingSubtitleTracks.clear();
            mCurrentState = STATE_IDLE;
            if (cleartargetstate) {
                mTargetState  = STATE_IDLE;
            }
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        if (isInPlaybackState() && mMediaController != null) {
            toggleMediaControlsVisiblity();
        }
        return false;
    }

    @Override
    public boolean onTrackballEvent(MotionEvent ev) {
        if (isInPlaybackState() && mMediaController != null) {
            toggleMediaControlsVisiblity();
        }
        return false;
    }

    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event)
    {
        boolean isKeyCodeSupported = keyCode != KeyEvent.KEYCODE_BACK &&
                                     keyCode != KeyEvent.KEYCODE_VOLUME_UP &&
                                     keyCode != KeyEvent.KEYCODE_VOLUME_DOWN &&
                                     keyCode != KeyEvent.KEYCODE_VOLUME_MUTE &&
                                     keyCode != KeyEvent.KEYCODE_MENU &&
                                     keyCode != KeyEvent.KEYCODE_CALL &&
                                     keyCode != KeyEvent.KEYCODE_ENDCALL;
        if (isInPlaybackState() && isKeyCodeSupported && mMediaController != null) {
            if (keyCode == KeyEvent.KEYCODE_HEADSETHOOK ||
                    keyCode == KeyEvent.KEYCODE_MEDIA_PLAY_PAUSE) {
                if (mMediaPlayer.isPlaying()) {
                    pause();
                    mMediaController.show();
                } else {
                    start();
                    mMediaController.hide();
                }
                return true;
            } else if (keyCode == KeyEvent.KEYCODE_MEDIA_PLAY) {
                if (!mMediaPlayer.isPlaying()) {
                    start();
                    mMediaController.hide();
                }
                return true;
            } else if (keyCode == KeyEvent.KEYCODE_MEDIA_STOP
                    || keyCode == KeyEvent.KEYCODE_MEDIA_PAUSE) {
                if (mMediaPlayer.isPlaying()) {
                    pause();
                    mMediaController.show();
                }
                return true;
            } else {
                toggleMediaControlsVisiblity();
            }
        }

        return super.onKeyDown(keyCode, event);
    }

    private void toggleMediaControlsVisiblity() {
        if (mMediaController.isShowing()) {
            mMediaController.hide();
        } else {
            mMediaController.show();
        }
    }

    @Override
    public void start() {
        if (isInPlaybackState()) {
            mMediaPlayer.start();
            mCurrentState = STATE_PLAYING;
        }
        mTargetState = STATE_PLAYING;
    }

    @Override
    public void pause() {
        if (isInPlaybackState()) {
            if (mMediaPlayer.isPlaying()) {
                mMediaPlayer.pause();
                mCurrentState = STATE_PAUSED;
            }
        }
        mTargetState = STATE_PAUSED;
    }

    public void suspend() {
        release(false);
    }

    public void resume() {
        openVideo();
    }

    @Override
    public int getDuration() {
        if (isInPlaybackState()) {
            return mMediaPlayer.getDuration();
        }

        return -1;
    }

    @Override
    public int getCurrentPosition() {
        if (isInPlaybackState()) {
            return mMediaPlayer.getCurrentPosition();
        }
        return 0;
    }

    @Override
    public void seekTo(int msec) {
        if (isInPlaybackState()) {
            mMediaPlayer.seekTo(msec);
            mSeekWhenPrepared = 0;
        } else {
            mSeekWhenPrepared = msec;
        }
    }

    @Override
    public boolean isPlaying() {
        return isInPlaybackState() && mMediaPlayer.isPlaying();
    }

    @Override
    public int getBufferPercentage() {
        if (mMediaPlayer != null) {
            return mCurrentBufferPercentage;
        }
        return 0;
    }

    private boolean isInPlaybackState() {
        return (mMediaPlayer != null &&
                mCurrentState != STATE_ERROR &&
                mCurrentState != STATE_IDLE &&
                mCurrentState != STATE_PREPARING);
    }

    @Override
    public boolean canPause() {
        return mCanPause;
    }

    @Override
    public boolean canSeekBackward() {
        return mCanSeekBack;
    }

    @Override
    public boolean canSeekForward() {
        return mCanSeekForward;
    }

    @Override
    public int getAudioSessionId() {
        if (mAudioSession == 0) {
            MediaPlayer foo = new MediaPlayer();
            mAudioSession = foo.getAudioSessionId();
            foo.release();
        }
        return mAudioSession;
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();

        if (mSubtitleWidget != null) {
            mSubtitleWidget.onAttachedToWindow();
        }
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();

        if (mSubtitleWidget != null) {
            mSubtitleWidget.onDetachedFromWindow();
        }
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);

        if (mSubtitleWidget != null) {
            measureAndLayoutSubtitleWidget();
        }
    }

    @Override
    public void draw(Canvas canvas) {
        super.draw(canvas);

        if (mSubtitleWidget != null) {
            final int saveCount = canvas.save();
            canvas.translate(getPaddingLeft(), getPaddingTop());
            mSubtitleWidget.draw(canvas);
            canvas.restoreToCount(saveCount);
        }
    }

    /**
     * Forces a measurement and layout pass for all overlaid views.
     *
     * @see #setSubtitleWidget(RenderingWidget)
     */
    private void measureAndLayoutSubtitleWidget() {
        final int width = getWidth() - getPaddingLeft() - getPaddingRight();
        final int height = getHeight() - getPaddingTop() - getPaddingBottom();

        mSubtitleWidget.setSize(width, height);
    }

    /** @hide */
    @Override
    public void setSubtitleWidget(RenderingWidget subtitleWidget) {
        if (mSubtitleWidget == subtitleWidget) {
            return;
        }

        final boolean attachedToWindow = isAttachedToWindow();
        if (mSubtitleWidget != null) {
            if (attachedToWindow) {
                mSubtitleWidget.onDetachedFromWindow();
            }

            mSubtitleWidget.setOnChangedListener(null);
        }

        mSubtitleWidget = subtitleWidget;

        if (subtitleWidget != null) {
            if (mSubtitlesChangedListener == null) {
                mSubtitlesChangedListener = new RenderingWidget.OnChangedListener() {
                    @Override
                    public void onChanged(RenderingWidget renderingWidget) {
                        invalidate();
                    }
                };
            }

            setWillNotDraw(false);
            subtitleWidget.setOnChangedListener(mSubtitlesChangedListener);

            if (attachedToWindow) {
                subtitleWidget.onAttachedToWindow();
                requestLayout();
            }
        } else {
            setWillNotDraw(true);
        }

        invalidate();
    }

    /** @hide */
    @Override
    public Looper getSubtitleLooper() {
        return Looper.getMainLooper();
    }
}
```

MediaPlayerControl
---
 
 通过该接口来打通MediaController以及VideoView
 
 ```java
 public interface MediaPlayerControl {
	void    start();
	void    pause();
	int     getDuration();
	int     getCurrentPosition();
	void    seekTo(int pos);
	boolean isPlaying();
	int     getBufferPercentage();
	boolean canPause();
	boolean canSeekBackward();
	boolean canSeekForward();

	/**
	 * Get the audio session id for the player used by this VideoView. This can be used to
	 * apply audio effects to the audio track of a video.
	 * @return The audio session, or 0 if there was an error.
	 */
	int     getAudioSessionId();
}
 ```
 
MediaController
---
 
- 简介            
    ```java
	/**
	* A view containing controls for a MediaPlayer. Typically contains the
	* buttons like "Play/Pause", "Rewind", "Fast Forward" and a progress
	* slider. It takes care of synchronizing the controls with the state
	* of the MediaPlayer.
	* <p>
	* The way to use this class is to instantiate it programatically.
	* The MediaController will create a default set of controls
	* and put them in a window floating above your application. Specifically,
	* the controls will float above the view specified with setAnchorView().
	* The window will disappear if left idle for three seconds and reappear
	* when the user touches the anchor view.
	* <p>
	* Functions like show() and hide() have no effect when MediaController
	* is created in an xml layout.
	* 
	* MediaController will hide and
	* show the buttons according to these rules:
	* <ul>
	* <li> The "previous" and "next" buttons are hidden until setPrevNextListeners()
	*   has been called
	* <li> The "previous" and "next" buttons are visible but disabled if
	*   setPrevNextListeners() was called with null listeners
	* <li> The "rewind" and "fastforward" buttons are shown unless requested
	*   otherwise by using the MediaController(Context, boolean) constructor
	*   with the boolean set to false
	* </ul>
	*/
	```

- 关系
    ```java
	public class MediaController extends FrameLayout
	```

- 成员
	```java
	// 一些控制功能的接口
	private MediaPlayerControl  mPlayer;
    private Context             mContext;
	// VideoView中调用setAnchorView()设置进来的View，MediaController显示的时候会感觉该AnchorView的位置进行显示
    private View                mAnchor;
	// MediaController最外层的根布局
    private View                mRoot;
	
	// 通过Window的方式来显示MediaController，MediaController是一个填充屏幕的布局，但是背景是透明的
    private WindowManager       mWindowManager;
    private Window              mWindow;
    private View                mDecor;
	// 理解为当前整个MediaController的布局
    private WindowManager.LayoutParams mDecorLayoutParams;
    private ProgressBar         mProgress;
    private TextView            mEndTime, mCurrentTime;
    private boolean             mShowing;
    private boolean             mDragging;
	// 默认自动消失的时间
    private static final int    sDefaultTimeout = 3000;
    private static final int    FADE_OUT = 1;
    private static final int    SHOW_PROGRESS = 2;
    private boolean             mUseFastForward;
    private boolean             mFromXml;
    private boolean             mListenersSet;
    private View.OnClickListener mNextListener, mPrevListener;
    StringBuilder               mFormatBuilder;
    Formatter                   mFormatter;
    private ImageButton         mPauseButton;
    private ImageButton         mFfwdButton;
    private ImageButton         mRewButton;
    private ImageButton         mNextButton;
    private ImageButton         mPrevButton;
	```
	
- 构造方法
    ```java
	public MediaController(Context context, AttributeSet attrs) {
        super(context, attrs);
        mRoot = this;
        mContext = context;
        mUseFastForward = true;
        mFromXml = true;
    }

    @Override
    public void onFinishInflate() {
        if (mRoot != null)
            initControllerView(mRoot);
    }
    
    public MediaController(Context context, boolean useFastForward) {
        super(context);
        mContext = context;
        mUseFastForward = useFastForward;
		// 创建该MediaController的布局
        initFloatingWindowLayout();
        initFloatingWindow();
    }
    
    public MediaController(Context context) {
        this(context, true);
    }
	```
	
	- initFloatingWindowLayout
	    ```java
		// Allocate and initialize the static parts of mDecorLayoutParams. Must
		// also call updateFloatingWindowLayout() to fill in the dynamic parts
		// (y and width) before mDecorLayoutParams can be used.
		private void initFloatingWindowLayout() {
			mDecorLayoutParams = new WindowManager.LayoutParams();
			WindowManager.LayoutParams p = mDecorLayoutParams;
			p.gravity = Gravity.TOP | Gravity.LEFT;
			p.height = LayoutParams.WRAP_CONTENT;
			p.x = 0;
			p.format = PixelFormat.TRANSLUCENT;
			p.type = WindowManager.LayoutParams.TYPE_APPLICATION_PANEL;
			p.flags |= WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM
					| WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
					| WindowManager.LayoutParams.FLAG_SPLIT_TOUCH;
			p.token = null;
			p.windowAnimations = 0; // android.R.style.DropDownAnimationDown;
		}
		```
		
	- initFloatingWindow
	    ```java
		private void initFloatingWindow() {
		    // Android内核剖析 中有介绍
			mWindowManager = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
			mWindow = PolicyManager.makeNewWindow(mContext);
			mWindow.setWindowManager(mWindowManager, null, null);
			mWindow.requestFeature(Window.FEATURE_NO_TITLE);
			// 通过WindowManager去add该Decor以及remove来实现MediaController的显示与隐藏
			mDecor = mWindow.getDecorView();
			mDecor.setOnTouchListener(mTouchListener);
			mWindow.setContentView(this);
			mWindow.setBackgroundDrawableResource(android.R.color.transparent);
			
			// While the media controller is up, the volume control keys should
			// affect the media stream type
			mWindow.setVolumeControlStream(AudioManager.STREAM_MUSIC);

			setFocusable(true);
			setFocusableInTouchMode(true);
			setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
			requestFocus();
		}
		```
		
	- mTouchListener        
		```java
		private OnTouchListener mTouchListener = new OnTouchListener() {
			public boolean onTouch(View v, MotionEvent event) {
				if (event.getAction() == MotionEvent.ACTION_DOWN) {
					if (mShowing) {
						hide();
					}
				}
				return false;
			}
		};
		```

	- setMediaPlayer
	    VideoView调用setMediaController的时候会调用到该方法
	    ```java
		public void setMediaPlayer(MediaPlayerControl player) {
			mPlayer = player;
			updatePausePlay();
		}
		```
		
	- setAnchorView
		VideoView调用setMediaController的时候会调用到该方法
	    ```java
		/**
		 * Set the view that acts as the anchor for the control view.
		 * This can for example be a VideoView, or your Activity's main view.
		 * When VideoView calls this method, it will use the VideoView's parent
		 * as the anchor.
		 * @param view The view to which to anchor the controller when it is visible.
		 */
		public void setAnchorView(View view) {
			if (mAnchor != null) {
				mAnchor.removeOnLayoutChangeListener(mLayoutChangeListener);
			}
			mAnchor = view;
			if (mAnchor != null) {
				mAnchor.addOnLayoutChangeListener(mLayoutChangeListener);
			}

			FrameLayout.LayoutParams frameParams = new FrameLayout.LayoutParams(
					ViewGroup.LayoutParams.MATCH_PARENT,
					ViewGroup.LayoutParams.MATCH_PARENT
			);

			removeAllViews();
			View v = makeControllerView();
			addView(v, frameParams);
		}
		```
		
		- mLayoutChangeListener
		    ```java
			// This is called whenever mAnchor's layout bound changes
			private OnLayoutChangeListener mLayoutChangeListener =
					new OnLayoutChangeListener() {
				public void onLayoutChange(View v, int left, int top, int right,
						int bottom, int oldLeft, int oldTop, int oldRight,
						int oldBottom) {
					// 更新布局
					updateFloatingWindowLayout();
					if (mShowing) {
						mWindowManager.updateViewLayout(mDecor, mDecorLayoutParams);
					}
				}
			};
			```

			- updateFloatingWindowLayout
			    ```java
				// Update the dynamic parts of mDecorLayoutParams
				// Must be called with mAnchor != NULL.
				private void updateFloatingWindowLayout() {
					int [] anchorPos = new int[2];
					mAnchor.getLocationOnScreen(anchorPos);

					// we need to know the size of the controller so we can properly position it
					// within its space
					mDecor.measure(MeasureSpec.makeMeasureSpec(mAnchor.getWidth(), MeasureSpec.AT_MOST),
							MeasureSpec.makeMeasureSpec(mAnchor.getHeight(), MeasureSpec.AT_MOST));

					WindowManager.LayoutParams p = mDecorLayoutParams;
					p.width = mAnchor.getWidth();
					p.x = anchorPos[0] + (mAnchor.getWidth() - p.width) / 2;
					p.y = anchorPos[1] + mAnchor.getHeight() - mDecor.getMeasuredHeight();
				}
				```
				
		- makeControllerView
		    ```java
			/**
			 * Create the view that holds the widgets that control playback.
			 * Derived classes can override this to create their own.
			 * @return The controller view.
			 * @hide This doesn't work as advertised
			 */
			protected View makeControllerView() {
				LayoutInflater inflate = (LayoutInflater) mContext.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
				mRoot = inflate.inflate(com.android.internal.R.layout.media_controller, null);
				// 对Controller中的一些按钮、功能进行事件设置
				initControllerView(mRoot);

				return mRoot;
			}
			```
			
	- touch事件处理
	    ```java
		@Override
		public boolean onTouchEvent(MotionEvent event) {
			show(sDefaultTimeout);
			return true;
		}
		```

	- 进度的处理
		- seekBar的处理
			```java
			// There are two scenarios that can trigger the seekbar listener to trigger:
			//
			// The first is the user using the touchpad to adjust the posititon of the
			// seekbar's thumb. In this case onStartTrackingTouch is called followed by
			// a number of onProgressChanged notifications, concluded by onStopTrackingTouch.
			// We're setting the field "mDragging" to true for the duration of the dragging
			// session to avoid jumps in the position in case of ongoing playback.
			//
			// The second scenario involves the user operating the scroll ball, in this
			// case there WON'T BE onStartTrackingTouch/onStopTrackingTouch notifications,
			// we will simply apply the updated position without suspending regular updates.
			private OnSeekBarChangeListener mSeekListener = new OnSeekBarChangeListener() {
				public void onStartTrackingTouch(SeekBar bar) {
					show(3600000);

					mDragging = true;

					// By removing these pending progress messages we make sure
					// that a) we won't update the progress while the user adjusts
					// the seekbar and b) once the user is done dragging the thumb
					// we will post one of these messages to the queue again and
					// this ensures that there will be exactly one message queued up.
					mHandler.removeMessages(SHOW_PROGRESS);
				}

				public void onProgressChanged(SeekBar bar, int progress, boolean fromuser) {
					if (!fromuser) {
						// We're not interested in programmatically generated changes to
						// the progress bar's position.
						return;
					}

					long duration = mPlayer.getDuration();
					long newposition = (duration * progress) / 1000L;
					mPlayer.seekTo( (int) newposition);
					if (mCurrentTime != null)
						mCurrentTime.setText(stringForTime( (int) newposition));
				}

				public void onStopTrackingTouch(SeekBar bar) {
					mDragging = false;
					setProgress();
					updatePausePlay();
					show(sDefaultTimeout);

					// Ensure that progress is properly updated in the future,
					// the call to show() does not guarantee this because it is a
					// no-op if we are already showing.
					mHandler.sendEmptyMessage(SHOW_PROGRESS);
				}
			};
			```
			
		- SetProgress
            ```java
			private int setProgress() {
				if (mPlayer == null || mDragging) {
					return 0;
				}
				int position = mPlayer.getCurrentPosition();
				int duration = mPlayer.getDuration();
				if (mProgress != null) {
					if (duration > 0) {
						// use long to avoid overflow
						long pos = 1000L * position / duration;
						mProgress.setProgress( (int) pos);
					}
					int percent = mPlayer.getBufferPercentage();
					mProgress.setSecondaryProgress(percent * 10);
				}

				if (mEndTime != null)
					mEndTime.setText(stringForTime(duration));
				if (mCurrentTime != null)
					mCurrentTime.setText(stringForTime(position));

				return position;
			}
			```
			
		- show
        	```java
        	/**
             * Show the controller on screen. It will go away
             * automatically after 'timeout' milliseconds of inactivity.
             * @param timeout The timeout in milliseconds. Use 0 to show
             * the controller until hide() is called.
             */
            public void show(int timeout) {
                if (!mShowing && mAnchor != null) {
        			// 先去设置一下进度
                    setProgress();
                    if (mPauseButton != null) {
                        mPauseButton.requestFocus();
                    }
                    disableUnsupportedButtons();
                    updateFloatingWindowLayout();
                    mWindowManager.addView(mDecor, mDecorLayoutParams);
                    mShowing = true;
                }
                updatePausePlay();
                
                // cause the progress bar to be updated even if mShowing
                // was already true.  This happens, for example, if we're
                // paused with the progress bar showing the user hits play.
        		// 发送定期更新进度的消息
                mHandler.sendEmptyMessage(SHOW_PROGRESS);
        
                Message msg = mHandler.obtainMessage(FADE_OUT);
                if (timeout != 0) {
                    mHandler.removeMessages(FADE_OUT);
                    mHandler.sendMessageDelayed(msg, timeout);
                }
            }
        	```

    	- hide
    	    ```java
    		/**
    		 * Remove the controller from the screen.
    		 */
    		public void hide() {
    			if (mAnchor == null)
    				return;
    
    			if (mShowing) {
    				try {
    					// 移除定期更新消息
    					mHandler.removeMessages(SHOW_PROGRESS);
    					mWindowManager.removeView(mDecor);
    				} catch (IllegalArgumentException ex) {
    					Log.w("MediaController", "already removed");
    				}
    				mShowing = false;
    			}
    		}
    		```
	
上源码
===

```java
/*
 * Copyright (C) 2006 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package android.widget;

import android.content.Context;
import android.graphics.PixelFormat;
import android.media.AudioManager;
import android.os.Handler;
import android.os.Message;
import android.util.AttributeSet;
import android.util.Log;
import android.view.Gravity;
import android.view.KeyEvent;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;
import android.view.Window;
import android.view.WindowManager;
import android.view.accessibility.AccessibilityEvent;
import android.view.accessibility.AccessibilityNodeInfo;
import android.widget.SeekBar.OnSeekBarChangeListener;

import com.android.internal.policy.PolicyManager;

import java.util.Formatter;
import java.util.Locale;

/**
 * A view containing controls for a MediaPlayer. Typically contains the
 * buttons like "Play/Pause", "Rewind", "Fast Forward" and a progress
 * slider. It takes care of synchronizing the controls with the state
 * of the MediaPlayer.
 * <p>
 * The way to use this class is to instantiate it programatically.
 * The MediaController will create a default set of controls
 * and put them in a window floating above your application. Specifically,
 * the controls will float above the view specified with setAnchorView().
 * The window will disappear if left idle for three seconds and reappear
 * when the user touches the anchor view.
 * <p>
 * Functions like show() and hide() have no effect when MediaController
 * is created in an xml layout.
 * 
 * MediaController will hide and
 * show the buttons according to these rules:
 * <ul>
 * <li> The "previous" and "next" buttons are hidden until setPrevNextListeners()
 *   has been called
 * <li> The "previous" and "next" buttons are visible but disabled if
 *   setPrevNextListeners() was called with null listeners
 * <li> The "rewind" and "fastforward" buttons are shown unless requested
 *   otherwise by using the MediaController(Context, boolean) constructor
 *   with the boolean set to false
 * </ul>
 */
public class MediaController extends FrameLayout {

    private MediaPlayerControl  mPlayer;
    private Context             mContext;
    private View                mAnchor;
    private View                mRoot;
    private WindowManager       mWindowManager;
    private Window              mWindow;
    private View                mDecor;
    private WindowManager.LayoutParams mDecorLayoutParams;
    private ProgressBar         mProgress;
    private TextView            mEndTime, mCurrentTime;
    private boolean             mShowing;
    private boolean             mDragging;
    private static final int    sDefaultTimeout = 3000;
    private static final int    FADE_OUT = 1;
    private static final int    SHOW_PROGRESS = 2;
    private boolean             mUseFastForward;
    private boolean             mFromXml;
    private boolean             mListenersSet;
    private View.OnClickListener mNextListener, mPrevListener;
    StringBuilder               mFormatBuilder;
    Formatter                   mFormatter;
    private ImageButton         mPauseButton;
    private ImageButton         mFfwdButton;
    private ImageButton         mRewButton;
    private ImageButton         mNextButton;
    private ImageButton         mPrevButton;

    public MediaController(Context context, AttributeSet attrs) {
        super(context, attrs);
        mRoot = this;
        mContext = context;
        mUseFastForward = true;
        mFromXml = true;
    }

    @Override
    public void onFinishInflate() {
        if (mRoot != null)
            initControllerView(mRoot);
    }

    public MediaController(Context context, boolean useFastForward) {
        super(context);
        mContext = context;
        mUseFastForward = useFastForward;
        initFloatingWindowLayout();
        initFloatingWindow();
    }

    public MediaController(Context context) {
        this(context, true);
    }

    private void initFloatingWindow() {
        mWindowManager = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        mWindow = PolicyManager.makeNewWindow(mContext);
        mWindow.setWindowManager(mWindowManager, null, null);
        mWindow.requestFeature(Window.FEATURE_NO_TITLE);
        mDecor = mWindow.getDecorView();
        mDecor.setOnTouchListener(mTouchListener);
        mWindow.setContentView(this);
        mWindow.setBackgroundDrawableResource(android.R.color.transparent);
        
        // While the media controller is up, the volume control keys should
        // affect the media stream type
        mWindow.setVolumeControlStream(AudioManager.STREAM_MUSIC);

        setFocusable(true);
        setFocusableInTouchMode(true);
        setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        requestFocus();
    }

    // Allocate and initialize the static parts of mDecorLayoutParams. Must
    // also call updateFloatingWindowLayout() to fill in the dynamic parts
    // (y and width) before mDecorLayoutParams can be used.
    private void initFloatingWindowLayout() {
        mDecorLayoutParams = new WindowManager.LayoutParams();
        WindowManager.LayoutParams p = mDecorLayoutParams;
        p.gravity = Gravity.TOP | Gravity.LEFT;
        p.height = LayoutParams.WRAP_CONTENT;
        p.x = 0;
        p.format = PixelFormat.TRANSLUCENT;
        p.type = WindowManager.LayoutParams.TYPE_APPLICATION_PANEL;
        p.flags |= WindowManager.LayoutParams.FLAG_ALT_FOCUSABLE_IM
                | WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
                | WindowManager.LayoutParams.FLAG_SPLIT_TOUCH;
        p.token = null;
        p.windowAnimations = 0; // android.R.style.DropDownAnimationDown;
    }

    // Update the dynamic parts of mDecorLayoutParams
    // Must be called with mAnchor != NULL.
    private void updateFloatingWindowLayout() {
        int [] anchorPos = new int[2];
        mAnchor.getLocationOnScreen(anchorPos);

        // we need to know the size of the controller so we can properly position it
        // within its space
        mDecor.measure(MeasureSpec.makeMeasureSpec(mAnchor.getWidth(), MeasureSpec.AT_MOST),
                MeasureSpec.makeMeasureSpec(mAnchor.getHeight(), MeasureSpec.AT_MOST));

        WindowManager.LayoutParams p = mDecorLayoutParams;
        p.width = mAnchor.getWidth();
        p.x = anchorPos[0] + (mAnchor.getWidth() - p.width) / 2;
        p.y = anchorPos[1] + mAnchor.getHeight() - mDecor.getMeasuredHeight();
    }

    // This is called whenever mAnchor's layout bound changes
    private OnLayoutChangeListener mLayoutChangeListener =
            new OnLayoutChangeListener() {
        public void onLayoutChange(View v, int left, int top, int right,
                int bottom, int oldLeft, int oldTop, int oldRight,
                int oldBottom) {
            updateFloatingWindowLayout();
            if (mShowing) {
                mWindowManager.updateViewLayout(mDecor, mDecorLayoutParams);
            }
        }
    };

    private OnTouchListener mTouchListener = new OnTouchListener() {
        public boolean onTouch(View v, MotionEvent event) {
            if (event.getAction() == MotionEvent.ACTION_DOWN) {
                if (mShowing) {
                    hide();
                }
            }
            return false;
        }
    };
    
    public void setMediaPlayer(MediaPlayerControl player) {
        mPlayer = player;
        updatePausePlay();
    }

    /**
     * Set the view that acts as the anchor for the control view.
     * This can for example be a VideoView, or your Activity's main view.
     * When VideoView calls this method, it will use the VideoView's parent
     * as the anchor.
     * @param view The view to which to anchor the controller when it is visible.
     */
    public void setAnchorView(View view) {
        if (mAnchor != null) {
            mAnchor.removeOnLayoutChangeListener(mLayoutChangeListener);
        }
        mAnchor = view;
        if (mAnchor != null) {
            mAnchor.addOnLayoutChangeListener(mLayoutChangeListener);
        }

        FrameLayout.LayoutParams frameParams = new FrameLayout.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT
        );

        removeAllViews();
        View v = makeControllerView();
        addView(v, frameParams);
    }

    /**
     * Create the view that holds the widgets that control playback.
     * Derived classes can override this to create their own.
     * @return The controller view.
     * @hide This doesn't work as advertised
     */
    protected View makeControllerView() {
        LayoutInflater inflate = (LayoutInflater) mContext.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        mRoot = inflate.inflate(com.android.internal.R.layout.media_controller, null);

        initControllerView(mRoot);

        return mRoot;
    }

    private void initControllerView(View v) {
        mPauseButton = (ImageButton) v.findViewById(com.android.internal.R.id.pause);
        if (mPauseButton != null) {
            mPauseButton.requestFocus();
            mPauseButton.setOnClickListener(mPauseListener);
        }

        mFfwdButton = (ImageButton) v.findViewById(com.android.internal.R.id.ffwd);
        if (mFfwdButton != null) {
            mFfwdButton.setOnClickListener(mFfwdListener);
            if (!mFromXml) {
                mFfwdButton.setVisibility(mUseFastForward ? View.VISIBLE : View.GONE);
            }
        }

        mRewButton = (ImageButton) v.findViewById(com.android.internal.R.id.rew);
        if (mRewButton != null) {
            mRewButton.setOnClickListener(mRewListener);
            if (!mFromXml) {
                mRewButton.setVisibility(mUseFastForward ? View.VISIBLE : View.GONE);
            }
        }

        // By default these are hidden. They will be enabled when setPrevNextListeners() is called 
        mNextButton = (ImageButton) v.findViewById(com.android.internal.R.id.next);
        if (mNextButton != null && !mFromXml && !mListenersSet) {
            mNextButton.setVisibility(View.GONE);
        }
        mPrevButton = (ImageButton) v.findViewById(com.android.internal.R.id.prev);
        if (mPrevButton != null && !mFromXml && !mListenersSet) {
            mPrevButton.setVisibility(View.GONE);
        }

        mProgress = (ProgressBar) v.findViewById(com.android.internal.R.id.mediacontroller_progress);
        if (mProgress != null) {
            if (mProgress instanceof SeekBar) {
                SeekBar seeker = (SeekBar) mProgress;
                seeker.setOnSeekBarChangeListener(mSeekListener);
            }
            mProgress.setMax(1000);
        }

        mEndTime = (TextView) v.findViewById(com.android.internal.R.id.time);
        mCurrentTime = (TextView) v.findViewById(com.android.internal.R.id.time_current);
        mFormatBuilder = new StringBuilder();
        mFormatter = new Formatter(mFormatBuilder, Locale.getDefault());

        installPrevNextListeners();
    }

    /**
     * Show the controller on screen. It will go away
     * automatically after 3 seconds of inactivity.
     */
    public void show() {
        show(sDefaultTimeout);
    }

    /**
     * Disable pause or seek buttons if the stream cannot be paused or seeked.
     * This requires the control interface to be a MediaPlayerControlExt
     */
    private void disableUnsupportedButtons() {
        try {
            if (mPauseButton != null && !mPlayer.canPause()) {
                mPauseButton.setEnabled(false);
            }
            if (mRewButton != null && !mPlayer.canSeekBackward()) {
                mRewButton.setEnabled(false);
            }
            if (mFfwdButton != null && !mPlayer.canSeekForward()) {
                mFfwdButton.setEnabled(false);
            }
        } catch (IncompatibleClassChangeError ex) {
            // We were given an old version of the interface, that doesn't have
            // the canPause/canSeekXYZ methods. This is OK, it just means we
            // assume the media can be paused and seeked, and so we don't disable
            // the buttons.
        }
    }
    
    /**
     * Show the controller on screen. It will go away
     * automatically after 'timeout' milliseconds of inactivity.
     * @param timeout The timeout in milliseconds. Use 0 to show
     * the controller until hide() is called.
     */
    public void show(int timeout) {
        if (!mShowing && mAnchor != null) {
            setProgress();
            if (mPauseButton != null) {
                mPauseButton.requestFocus();
            }
            disableUnsupportedButtons();
            updateFloatingWindowLayout();
            mWindowManager.addView(mDecor, mDecorLayoutParams);
            mShowing = true;
        }
        updatePausePlay();
        
        // cause the progress bar to be updated even if mShowing
        // was already true.  This happens, for example, if we're
        // paused with the progress bar showing the user hits play.
        mHandler.sendEmptyMessage(SHOW_PROGRESS);

        Message msg = mHandler.obtainMessage(FADE_OUT);
        if (timeout != 0) {
            mHandler.removeMessages(FADE_OUT);
            mHandler.sendMessageDelayed(msg, timeout);
        }
    }
    
    public boolean isShowing() {
        return mShowing;
    }

    /**
     * Remove the controller from the screen.
     */
    public void hide() {
        if (mAnchor == null)
            return;

        if (mShowing) {
            try {
                mHandler.removeMessages(SHOW_PROGRESS);
                mWindowManager.removeView(mDecor);
            } catch (IllegalArgumentException ex) {
                Log.w("MediaController", "already removed");
            }
            mShowing = false;
        }
    }

    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            int pos;
            switch (msg.what) {
                case FADE_OUT:
                    hide();
                    break;
                case SHOW_PROGRESS:
                    pos = setProgress();
                    if (!mDragging && mShowing && mPlayer.isPlaying()) {
                        msg = obtainMessage(SHOW_PROGRESS);
                        sendMessageDelayed(msg, 1000 - (pos % 1000));
                    }
                    break;
            }
        }
    };

    private String stringForTime(int timeMs) {
        int totalSeconds = timeMs / 1000;

        int seconds = totalSeconds % 60;
        int minutes = (totalSeconds / 60) % 60;
        int hours   = totalSeconds / 3600;

        mFormatBuilder.setLength(0);
        if (hours > 0) {
            return mFormatter.format("%d:%02d:%02d", hours, minutes, seconds).toString();
        } else {
            return mFormatter.format("%02d:%02d", minutes, seconds).toString();
        }
    }

    private int setProgress() {
        if (mPlayer == null || mDragging) {
            return 0;
        }
        int position = mPlayer.getCurrentPosition();
        int duration = mPlayer.getDuration();
        if (mProgress != null) {
            if (duration > 0) {
                // use long to avoid overflow
                long pos = 1000L * position / duration;
                mProgress.setProgress( (int) pos);
            }
            int percent = mPlayer.getBufferPercentage();
            mProgress.setSecondaryProgress(percent * 10);
        }

        if (mEndTime != null)
            mEndTime.setText(stringForTime(duration));
        if (mCurrentTime != null)
            mCurrentTime.setText(stringForTime(position));

        return position;
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        show(sDefaultTimeout);
        return true;
    }

    @Override
    public boolean onTrackballEvent(MotionEvent ev) {
        show(sDefaultTimeout);
        return false;
    }

    @Override
    public boolean dispatchKeyEvent(KeyEvent event) {
        int keyCode = event.getKeyCode();
        final boolean uniqueDown = event.getRepeatCount() == 0
                && event.getAction() == KeyEvent.ACTION_DOWN;
        if (keyCode ==  KeyEvent.KEYCODE_HEADSETHOOK
                || keyCode == KeyEvent.KEYCODE_MEDIA_PLAY_PAUSE
                || keyCode == KeyEvent.KEYCODE_SPACE) {
            if (uniqueDown) {
                doPauseResume();
                show(sDefaultTimeout);
                if (mPauseButton != null) {
                    mPauseButton.requestFocus();
                }
            }
            return true;
        } else if (keyCode == KeyEvent.KEYCODE_MEDIA_PLAY) {
            if (uniqueDown && !mPlayer.isPlaying()) {
                mPlayer.start();
                updatePausePlay();
                show(sDefaultTimeout);
            }
            return true;
        } else if (keyCode == KeyEvent.KEYCODE_MEDIA_STOP
                || keyCode == KeyEvent.KEYCODE_MEDIA_PAUSE) {
            if (uniqueDown && mPlayer.isPlaying()) {
                mPlayer.pause();
                updatePausePlay();
                show(sDefaultTimeout);
            }
            return true;
        } else if (keyCode == KeyEvent.KEYCODE_VOLUME_DOWN
                || keyCode == KeyEvent.KEYCODE_VOLUME_UP
                || keyCode == KeyEvent.KEYCODE_VOLUME_MUTE
                || keyCode == KeyEvent.KEYCODE_CAMERA) {
            // don't show the controls for volume adjustment
            return super.dispatchKeyEvent(event);
        } else if (keyCode == KeyEvent.KEYCODE_BACK || keyCode == KeyEvent.KEYCODE_MENU) {
            if (uniqueDown) {
                hide();
            }
            return true;
        }

        show(sDefaultTimeout);
        return super.dispatchKeyEvent(event);
    }

    private View.OnClickListener mPauseListener = new View.OnClickListener() {
        public void onClick(View v) {
            doPauseResume();
            show(sDefaultTimeout);
        }
    };

    private void updatePausePlay() {
        if (mRoot == null || mPauseButton == null)
            return;

        if (mPlayer.isPlaying()) {
            mPauseButton.setImageResource(com.android.internal.R.drawable.ic_media_pause);
        } else {
            mPauseButton.setImageResource(com.android.internal.R.drawable.ic_media_play);
        }
    }

    private void doPauseResume() {
        if (mPlayer.isPlaying()) {
            mPlayer.pause();
        } else {
            mPlayer.start();
        }
        updatePausePlay();
    }

    // There are two scenarios that can trigger the seekbar listener to trigger:
    //
    // The first is the user using the touchpad to adjust the posititon of the
    // seekbar's thumb. In this case onStartTrackingTouch is called followed by
    // a number of onProgressChanged notifications, concluded by onStopTrackingTouch.
    // We're setting the field "mDragging" to true for the duration of the dragging
    // session to avoid jumps in the position in case of ongoing playback.
    //
    // The second scenario involves the user operating the scroll ball, in this
    // case there WON'T BE onStartTrackingTouch/onStopTrackingTouch notifications,
    // we will simply apply the updated position without suspending regular updates.
    private OnSeekBarChangeListener mSeekListener = new OnSeekBarChangeListener() {
        public void onStartTrackingTouch(SeekBar bar) {
            show(3600000);

            mDragging = true;

            // By removing these pending progress messages we make sure
            // that a) we won't update the progress while the user adjusts
            // the seekbar and b) once the user is done dragging the thumb
            // we will post one of these messages to the queue again and
            // this ensures that there will be exactly one message queued up.
            mHandler.removeMessages(SHOW_PROGRESS);
        }

        public void onProgressChanged(SeekBar bar, int progress, boolean fromuser) {
            if (!fromuser) {
                // We're not interested in programmatically generated changes to
                // the progress bar's position.
                return;
            }

            long duration = mPlayer.getDuration();
            long newposition = (duration * progress) / 1000L;
            mPlayer.seekTo( (int) newposition);
            if (mCurrentTime != null)
                mCurrentTime.setText(stringForTime( (int) newposition));
        }

        public void onStopTrackingTouch(SeekBar bar) {
            mDragging = false;
            setProgress();
            updatePausePlay();
            show(sDefaultTimeout);

            // Ensure that progress is properly updated in the future,
            // the call to show() does not guarantee this because it is a
            // no-op if we are already showing.
            mHandler.sendEmptyMessage(SHOW_PROGRESS);
        }
    };

    @Override
    public void setEnabled(boolean enabled) {
        if (mPauseButton != null) {
            mPauseButton.setEnabled(enabled);
        }
        if (mFfwdButton != null) {
            mFfwdButton.setEnabled(enabled);
        }
        if (mRewButton != null) {
            mRewButton.setEnabled(enabled);
        }
        if (mNextButton != null) {
            mNextButton.setEnabled(enabled && mNextListener != null);
        }
        if (mPrevButton != null) {
            mPrevButton.setEnabled(enabled && mPrevListener != null);
        }
        if (mProgress != null) {
            mProgress.setEnabled(enabled);
        }
        disableUnsupportedButtons();
        super.setEnabled(enabled);
    }

    @Override
    public void onInitializeAccessibilityEvent(AccessibilityEvent event) {
        super.onInitializeAccessibilityEvent(event);
        event.setClassName(MediaController.class.getName());
    }

    @Override
    public void onInitializeAccessibilityNodeInfo(AccessibilityNodeInfo info) {
        super.onInitializeAccessibilityNodeInfo(info);
        info.setClassName(MediaController.class.getName());
    }

    private View.OnClickListener mRewListener = new View.OnClickListener() {
        public void onClick(View v) {
            int pos = mPlayer.getCurrentPosition();
            pos -= 5000; // milliseconds
            mPlayer.seekTo(pos);
            setProgress();

            show(sDefaultTimeout);
        }
    };

    private View.OnClickListener mFfwdListener = new View.OnClickListener() {
        public void onClick(View v) {
            int pos = mPlayer.getCurrentPosition();
            pos += 15000; // milliseconds
            mPlayer.seekTo(pos);
            setProgress();

            show(sDefaultTimeout);
        }
    };

    private void installPrevNextListeners() {
        if (mNextButton != null) {
            mNextButton.setOnClickListener(mNextListener);
            mNextButton.setEnabled(mNextListener != null);
        }

        if (mPrevButton != null) {
            mPrevButton.setOnClickListener(mPrevListener);
            mPrevButton.setEnabled(mPrevListener != null);
        }
    }

    public void setPrevNextListeners(View.OnClickListener next, View.OnClickListener prev) {
        mNextListener = next;
        mPrevListener = prev;
        mListenersSet = true;

        if (mRoot != null) {
            installPrevNextListeners();
            
            if (mNextButton != null && !mFromXml) {
                mNextButton.setVisibility(View.VISIBLE);
            }
            if (mPrevButton != null && !mFromXml) {
                mPrevButton.setVisibility(View.VISIBLE);
            }
        }
    }

    public interface MediaPlayerControl {
        void    start();
        void    pause();
        int     getDuration();
        int     getCurrentPosition();
        void    seekTo(int pos);
        boolean isPlaying();
        int     getBufferPercentage();
        boolean canPause();
        boolean canSeekBackward();
        boolean canSeekForward();

        /**
         * Get the audio session id for the player used by this VideoView. This can be used to
         * apply audio effects to the audio track of a video.
         * @return The audio session, or 0 if there was an error.
         */
        int     getAudioSessionId();
    }
}
```
 
---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 
