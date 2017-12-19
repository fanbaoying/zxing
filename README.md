#### ZXing二维码扫描Demo

在APP开发中，常遇到二维码扫描功能和生成二维码的需求。Android大部分是集成了zxing这个开源项目的扫码功能。
[开源项目地址](https://github.com/zxing/zxing)
下面给大家介绍一下具体的集成步骤
#### 集成步骤

[原文链接](https://juejin.im/post/5a335b6a51882549a746423e)

#### 1.demo展示如下：
1.1demo首页

![demo首页](http://upload-images.jianshu.io/upload_images/2829694-dd8d85b131a41b07.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.2扫描界面

![扫描界面](http://upload-images.jianshu.io/upload_images/2829694-578dfcbf6bd7c494.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以根据需求修改，我实际项目中界面截图如下：

![实际项目截图](http://upload-images.jianshu.io/upload_images/2829694-0e6d611ff4207d53.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1.3生成二维码

![生成二维码](http://upload-images.jianshu.io/upload_images/2829694-3006805b22b670c7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2.引入文件
2.1 下载demo，拷贝demo中的com.google.zxing5个包和com.utils包引入到自己的项目中。

![src目录](http://upload-images.jianshu.io/upload_images/2829694-0860da9a85e701dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.2 拷贝本项目demo中的布局activity_scanner.xml和toolbar_scanner.xml

![布局文件](http://upload-images.jianshu.io/upload_images/2829694-599a465a07f4882d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.3 拷贝资源目录raw至本项目中，beep.ogg是扫描成功时的提示音。
![提示音文件](http://upload-images.jianshu.io/upload_images/2829694-e3be0eeede7bb9eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.4 拷贝或合并文件内容attrs.xml/colors.xml/ids.xml三个文件。

![资源文件](http://upload-images.jianshu.io/upload_images/2829694-e755cfe0a0f9120f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.5 build.gradle文件中添加引用
```
compile 'com.google.zxing:core:3.3.0'
```
2.6 修改R文件引用路径
修改以下4个文件中的R文件引用地址，引用本项目的R
```
//com.google替换成自己项目的包名即可
com.google.zxing.activity.CaptureActivity
com.google.zxing.decoding.CaptureActivityHandler
com.google.zxing.decoding.DecodeHandler
com.google.zxing.view.ViewfinderView
```
###3. 权限配置
3.1 AndroidManifest.xml中添加权限申请代码：
```
<uses-permission android:name="android.permission.INTERNET" /> <!-- 网络权限 -->
<uses-permission android:name="android.permission.VIBRATE" /> <!-- 震动权限 -->
<uses-permission android:name="android.permission.CAMERA" /> <!-- 摄像头权限 -->
<uses-feature android:name="android.hardware.camera.autofocus" /> <!-- 自动聚焦权限 -->
```
### 4. 功能实现
完成上述集成之后，通过调用CaptureActivity就可以实现扫码功能。
MainActivity源码部分：
```
public class MainActivity extends AppCompatActivity {

@BindView(R.id.openQrCodeScan)
Button openQrCodeScan;
@BindView(R.id.text)
EditText text;
@BindView(R.id.CreateQrCode)
Button CreateQrCode;
@BindView(R.id.QrCode)
ImageView QrCode;
@BindView(R.id.qrCodeText)
TextView qrCodeText;

//打开扫描界面请求码
private int REQUEST_CODE = 0x01;
//扫描成功返回码
private int RESULT_OK = 0xA1;

@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
ButterKnife.bind(this);
}

@OnClick({R.id.openQrCodeScan, R.id.CreateQrCode})
public void onClick(View view) {
switch (view.getId()) {
case R.id.openQrCodeScan:
//打开二维码扫描界面
if(CommonUtil.isCameraCanUse()){
Intent intent = new Intent(MainActivity.this, CaptureActivity.class);
startActivityForResult(intent, REQUEST_CODE);
}else{
Toast.makeText(this,"请打开此应用的摄像头权限！",Toast.LENGTH_SHORT).show();
}
break;
case R.id.CreateQrCode:
try {
//获取输入的文本信息
String str = text.getText().toString().trim();
if(str != null && !"".equals(str.trim())){
//根据输入的文本生成对应的二维码并且显示出来
Bitmap mBitmap = EncodingHandler.createQRCode(text.getText().toString(), 500);
if(mBitmap != null){
Toast.makeText(this,"二维码生成成功！",Toast.LENGTH_SHORT).show();
QrCode.setImageBitmap(mBitmap);
}
}else{
Toast.makeText(this,"文本信息不能为空！",Toast.LENGTH_SHORT).show();
}
} catch (WriterException e) {
e.printStackTrace();
}
break;
}
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
super.onActivityResult(requestCode, resultCode, data);
//扫描结果回调
if (resultCode == RESULT_OK) { //RESULT_OK = -1
Bundle bundle = data.getExtras();
String scanResult = bundle.getString("qr_scan_result");
//将扫描出的信息显示出来
qrCodeText.setText(scanResult);
}
}
}

```
### 5. 源码分析
5.1打开二维码扫描界面
```
//打开二维码扫描界面
if(CommonUtil.isCameraCanUse()){
Intent intent = new Intent(MainActivity.this, CaptureActivity.class);
startActivityForResult(intent, REQUEST_CODE);
}else{
Toast.makeText(this,"请打开此应用的摄像头权限！",Toast.LENGTH_SHORT).show();
}
```
5.2 根据输入的文本生成对应的二维码并且显示出来
```
try {
//获取输入的文本信息
String str = text.getText().toString().trim();
if(str != null && !"".equals(str.trim())){
//根据输入的文本生成对应的二维码并且显示出来
Bitmap mBitmap = EncodingHandler.createQRCode(text.getText().toString(), 500);
if(mBitmap != null){
Toast.makeText(this,"二维码生成成功！",Toast.LENGTH_SHORT).show();
QrCode.setImageBitmap(mBitmap);
}
}else{
Toast.makeText(this,"文本信息不能为空！",Toast.LENGTH_SHORT).show();
}
} catch (WriterException e) {
e.printStackTrace();
}
```
5.3 扫描结果回调
```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
super.onActivityResult(requestCode, resultCode, data);
//扫描结果回调
if (resultCode == RESULT_OK) { //RESULT_OK = -1
Bundle bundle = data.getExtras();
String scanResult = bundle.getString("qr_scan_result");
//将扫描出的信息显示出来
qrCodeText.setText(scanResult);
}
}
```

> 希望可以帮助大家
> 如果哪里有什么不对或者不足的地方，还望读者多多提意见或建议
> Android技术交流群：591625129

![公众号](http://upload-images.jianshu.io/upload_images/2829694-48307b4d71bc5800.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)
