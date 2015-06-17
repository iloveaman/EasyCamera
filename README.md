
���Ȩ�ޣ�
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
     * �ж��Ƿ���Ԥ��״̬
     */
    private boolean previewIsRunning = false;

    private EasyCamera mEasyCamera;
    private EasyCamera.CameraActions mCameraActions;


    @OnClick({R.id.button1,R.id.button2,R.id.button3})
    public void openCamera(View view){
        EasyCamera.PictureCallback mPictureCallback = new EasyCamera.PictureCallback() {
            @Override
            public void onPictureTaken(byte[] data, EasyCamera.CameraActions actions) {
                // ֹͣԤ��
                mEasyCamera.stopPreview();
                try {
                    // ��ʱ1000s�����ĸ������
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // �洢ͼƬ�Ĳ���
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

                // ����Ԥ��
                EasyCamera.Callbacks.create().withRestartPreviewAfterCallbacks(true);
                try {
                    mEasyCamera.startPreview(mSurfaceHolder);
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        };
        // ������Ƭ�Ĳ���
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
                //��������������ͷ
                mEasyCamera = DefaultEasyCamera.open(Camera.CameraInfo.CAMERA_FACING_BACK);
                // ���������ת�Ƕ�
                //  mEasyCamera.setDisplayOrientation(90);
                //��������ʾ���Ԥ��
                WindowManager manager = (WindowManager) MainActivity.this.getSystemService(Context.WINDOW_SERVICE);
                mEasyCamera.alignCameraAndDisplayOrientation(manager);
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
                if (!previewIsRunning && mEasyCamera != null) {
                    try {
                        // ����Ԥ��
                        mCameraActions = mEasyCamera.startPreview(holder);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    previewIsRunning = true;
                }
            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {
                Log.d("TAG","surfaceDestroyedֹͣ���ͷ�");
                // ֹͣԤ�����ͷ�
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
 