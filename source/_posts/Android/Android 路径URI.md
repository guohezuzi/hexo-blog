# Android 路径URI

1. assets目录
   
   - 对于webview才能用: **file:///andriod_asset/....**
   
   - 项目中使用AssetManager获取文件内容

2. res/raw目录
   
   使用 **android.resource://包名(类型pers.zuzi.android)/+R.raw....**

3. 系统目录URI
   
   对于系统磁盘:Uri.fromFile(new File(Environment.getExternalStorageDirectory().getPath()+"/"+"系统路径"))

4. 其他app
   
   使用content://
