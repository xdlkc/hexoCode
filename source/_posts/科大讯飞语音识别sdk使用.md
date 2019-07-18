---
title: 科大讯飞语音识别sdk使用
date: 2018-04-29 01:32:42
tags:
categories: android
---

## 注册并获取APPID

先到[讯飞开放平台](http://www.xfyun.cn/)创建一个包含语音识别模块的应用,并复制APPID

<!--more-->

![](http://otzriul1v.bkt.clouddn.com/18-4-29/62809145.jpg)

## 下载SDK

![](http://otzriul1v.bkt.clouddn.com/18-4-29/95488964.jpg)

![](http://otzriul1v.bkt.clouddn.com/18-4-29/74098769.jpg)

1. 下载后会得到一个压缩包，将./libs文件夹下的jar文件拷贝到工程目录/app/libs下并通过Android Studio将jar依赖加入进去
2. 将libs下的armeabi文件夹内的so文件拷贝到工程目录/app/main/jniLibs/armeabi下，这里如果没有jniLibs文件夹请手动创建（之所以不拷贝其他架构文件夹的so文件，是因为我加入了地图SDK的so，如果将其他so文件加入的话会崩溃，所以只加了一个）
3. 讯飞还提供了一个语音输入的录音框，将压缩包内assets文件夹里面的内容拷贝到工程目录/app/src/main/assets文件夹下（若assets文件夹不存在，请手动创建）

## 使用录音框及语音识别

1. 首先在对应的Activity中构造识别工具(APPID替换为你的APPID):

   ```java
   SpeechUtility.createUtility(this, SpeechConstant.APPID +"=APPID");
   ```

2. 创建相应的监听器（gson通过Android Studio add library dependency添加） 

   ```java
   initListener = new InitListener() {
     @Override
     public void onInit(int i) {

     }
   };
   // 语音识别返回
   recognizerDialogListener = new RecognizerDialogListener() {
     @Override
     public void onResult(RecognizerResult recognizerResult, boolean b) {
       // 由于返回的结果是一个json格式字符串，所以这里我们利用谷歌提供的gson来将其解析到类中
       Gson gson = new Gson();
       SpeechDo speechDo = gson.fromJson(recognizerResult.getResultString(),SpeechDo.class);
       // SpeechDo,SpeechV2,SpeechV3均为自己创建的存储对象，具体属性参照讯飞语音识别返回的结果
       for (SpeechDo.SpeechV2 speechV2 : speechDo.ws){
         for (SpeechDo.SpeechV3 speechV3 : speechV2.cw){
           speechBuilder.append(speechV3.w);
         }
       }
       // 是否为识别的最后一个单词，是的话进行下一步处理
       if (speechDo.ls){
         
         speechToText();
       }
     }
     @Override
     public void onError(SpeechError speechError) {

     }
   };
   ```

3. 在合适的位置（如按钮点击事件创建录音框）

   ```java
   ///1.创建 RecognizerDialog 对象
   RecognizerDialog mDialog = new RecognizerDialog(this, initListener);
   //若要将 RecognizerDialog 用于语义理解，必须添加以下参数设置，设置之后 onResult 回调返回将是语义理解的结果
   // mDialog.setParameter("asr_sch", "1");
   // mDialog.setParameter("nlp_version", "3.0");
   // 设置中文
   mDialog.setParameter(SpeechConstant. LANGUAGE, "zh_cn" );
   mDialog.setParameter(SpeechConstant. ACCENT, "mandarin" );
   //3.设置回调接口
   mDialog.setListener( recognizerDialogListener );
   //4.显示 dialog，接收语音输入
   mDialog.show();
   ```

   ​