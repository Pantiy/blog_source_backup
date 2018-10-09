---
title: 从Uri获取文件路径
date: 2017-11-26 19:13:55
tags:
---
*作者：Pantiy ，转载请注明出处*

> 在 Android 的开发中，当我们需要去选择一个本地文件并对其操作时往往需要获取其在手机上的路径。通常会使用「隐式 Intent」 的方式去调用某 Activity，通常情况下这类 Activity 会返回给我们一个 Uri 用以获取文件所在路径。本文主要讲述如何从 Uri 获取文件路径。

**一、什么是 Uri**

Uri 全称 Uniform Resource Identifier，中文翻译为「统一资源标识符」。通常为
**「协议://域名/目录/文件」**的格式。  

例如，“ http://pantiy.me/archives/ ”。  

"http" 表示协议，“pantiy.me” 表示域名，“archives” 表示该主机下的一个目录。  

**二、Android 里文件的 Uri**

Android 中访问文件的 Uri 有两种不同的形式，也可是说是两种不同的协议。  
1. file:// 协议
2. content:// 协议  

对于「file」协议的处理较为简单，代码如下：
```
public static String getPath(Context context, Uri uri) {
  if("file".equalsIgnoreCase(uri.getScheme())) {
    return uri.getPath();
  }
  return null;
}
```

对于「content」协议的处理则相对麻烦一些。"content://" 开头的 Uri 有好几种不同的类型，对应也有不同的处理方法，这就是其相对麻烦一些的原因。  

下面贴出代码：
```
public static String getPath(Context context, Uri uri) {

  //DocumentsProvider
  if (DucumentsContract.isDocumentUri(context, uri)){

    String documentId = DocumentsContract.getDocumentId(uri);

    if (isExternalStorageDocument(uri)) {
      //ExternalStorageProvider

      String[] splitResult = documentId.split(":");
      if ("primary".equalsIgnoreCase(splitResult[0])) {
        return Environment.getExternalStorageDirectory() + "/" + splitResult[1];
      }
    } else if (isDownloadsDocument(uri)) {
      //DownloadsProvider
      Uri newUri = ContentUri.withAppendedId(
        Uri.parse("content://downloads/public_downloads"),
        Long.valueOf(documentId));
        
        return getDataColumnIndexOrThrow(context, uri, null, null);  
    } else if (isMediaDocument(uri)) {
      //MediaProvider

      String[] splitResult = documentId.split(":");
      Uri newUri = null;

      if ("image".equalsIgnoreCase(splitResult[0])) {
        newUri = MediaStore.Image.Media.EXTERNAL_CONTENT_URI;
      } else if ("video".equalsIgnoreCase(splitResult[0])) {
        newUri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
      } else if ("audio".equalsIgnoreCase(splitResult[0])) {
        newUri = MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
      }

      return getDataColumnIndexOrThrow(context, uri, "_id=?",new String[] {splitResult[1]});  
    }
  } else if ("content".equalsIgnoreCase(uri.getScheme())) {
    //MediaStore
    return getDataColumnIndexOrThrow(context, uri, null, null);
  } else if ("file".equalsIgnoreCase(uri.getScheme())) {  
    //File
    return uri.getPath();
  }
  return null;
}

private static String getDataColumnIndexOrThrow(Context context, Uri uri, String selection, String[] selectionArgs) {
  String dataColumn = null;
  String data = "_data";
  String[] protection = new String[] {data};
  Cursor cursor = context.getContentResolver().query(uri, protection, selection, selectionArgs, null);
  if (cursor == null || cursor.getCount() == 0) {
    Log.i(TAG, "cursor is null or cursor's count is 0");
    return null;
  }
  cursor.moveToFirst();
  dataColumn = cursor.getString(cursor.getColumnIndexOrThrow(data);
  cursor.close();
  return dataColumn;
}

private static boolean isExternalStorageDocument(Uri uri) {
  return "com.android.externalstorage.documents".equals(uri.getAuthority());
}

private static boolean isDownloadsDocument(Uri uri) {
  return "com.android.providers.downloads.documents".equals(uri.getAuthority());
}

private static boolean isMediaDocument(Uri uri) {
  return "com.android.providers.media.documents".equals(uri.getAuthority());
}
```
