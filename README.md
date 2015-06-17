
添加权限：
<uses-permission android:name="android.permission.CAMERA"/>
<uses-feature android:name="android.hardware.camera" android:required="false"/>



public class MainActivity extends ActionBarActivity {

    @InjectView(R.id.button1)
    Button button1;
    @InjectView(R.id.button2)
    Button button2;
    @InjectView(R.id.button3)
    Button button3;
    @InjectView(R.id.surfaceView1)
    SurfaceView surfaceView1;

    SurfaceHolder mSurfaceHolder;



    /**
     * 判断是否在预览状态
     */
    private boolean previewIsRunning = false;

    private EasyCamera mEasyCamera;
    private EasyCamera.CameraActions mCameraActions;


    @OnClick({R.id.button1,R.id.button2,R.id.button3})
    public void openCamera(View view){
        EasyCamera.PictureCallback mPictureCallback = new EasyCamera.PictureCallback() {
            @Override
            public void onPictureTaken(byte[] data, EasyCamera.CameraActions actions) {
                // 停止预览
                mEasyCamera.stopPreview();
                try {
                    // 延时1000s，看的更加清楚
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 存储图片的操作
                FileOutputStream fos = null;
                try {
                    Log.d("TAG", getExternalFilesDir(Environment.DIRECTORY_PICTURES) + File.separator + "abc.jpg");
                    fos = new FileOutputStream(new File(getExternalFilesDir(Environment.DIRECTORY_PICTURES)+File.separator+"abc.jpg"));
                    fos.write(data);
                    fos.close();
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }

                // 开启预览
                EasyCamera.Callbacks.create().withRestartPreviewAfterCallbacks(true);
                try {
                    mEasyCamera.startPreview(mSurfaceHolder);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        };
        // 拍摄照片的操作
        mCameraActions.takePicture(EasyCamera.Callbacks.create().withJpegCallback(mPictureCallback));
    }


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.inject(this);
        mSurfaceHolder = surfaceView1.getHolder();
        mSurfaceHolder.addCallback(new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {
                //　开启后置摄像头
                mEasyCamera = DefaultEasyCamera.open(Camera.CameraInfo.CAMERA_FACING_BACK);
                // 设置相机翻转角度
                //  mEasyCamera.setDisplayOrientation(90);
                //　正常显示相机预览
                WindowManager manager = (WindowManager) MainActivity.this.getSystemService(Context.WINDOW_SERVICE);
                mEasyCamera.alignCameraAndDisplayOrientation(manager);
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
                if (!previewIsRunning && mEasyCamera != null) {
                    try {
                        // 开启预览
                        mCameraActions = mEasyCamera.startPreview(holder);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    previewIsRunning = true;
                }
            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {
                Log.d("TAG","surfaceDestroyed停止并释放");
                // 停止预览并释放
                mEasyCamera.stopPreview();
                //mEasyCamera.release();
                mEasyCamera.close();
                mEasyCamera = null;
                mCameraActions = null;
                previewIsRunning = false;
            }
        });

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {
            return true;
        }

        return super.onOptionsItemSelected(item);
    }
}
 