# ImagePicker
### Android自定义相册，完全仿微信UI，实现了拍照、图片选择（单选/多选）、 裁剪 、旋转、等功能.
</br>该项目参考了：
* (https://github.com/jeasonlzy/ImagePicker)<br>
## 演示
  ![image](https://github.com/superBlossom/ImagePicker/raw/master/screenshot/device-2017-03-17-091954.png)![image](https://github.com/superBlossom/ImagePicker/raw/master/screenshot/device-2017-03-17-092040.png)
## 1.用法
<br>对于studio的用户可以在项目gradle的根目录添加：
```Java
maven { url 'https://jitpack.io' }
```
<br>然后在app对应的gradle下添加:
```Java
compile 'com.github.superBlossom:ImageChooser:1.0'
```
## 2.功能和参数含义
<table>
  <tdead>
    <tr>
      <th align="center">配置参数</th>
      <th align="center">参数含义</th>
    </tr>
  </tdead>
  <tbody>
    <tr>
      <td align="center">multiMode</td>
      <td align="center">图片选着模式，单选/多选</td>
    </tr>
    <tr>
      <td align="center">selectLimit</td>
      <td align="center">多选限制数量，默认为9</td>
    </tr>
    <tr>
      <td align="center">showCamera</td>
      <td align="center">选择照片时是否显示拍照按钮</td>
    </tr>
    <tr>
      <td align="center">crop</td>
      <td align="center">是否允许裁剪（单选有效）</td>
    </tr>
    <tr>
      <td align="center">style</td>
      <td align="center">有裁剪时，裁剪框是矩形还是圆形</td>
    </tr>
    <tr>
      <td align="center">focusWidth</td>
      <td align="center">矩形裁剪框宽度（圆形自动取宽高最小值）</td>
    </tr>
    <tr>
      <td align="center">focusHeight</td>
      <td align="center">矩形裁剪框高度（圆形自动取宽高最小值）</td>
    </tr>
    <tr>
      <td align="center">outPutX</td>
      <td align="center">裁剪后需要保存的图片宽度</td>
    </tr>
    <tr>
      <td align="center">outPutY</td>
      <td align="center">裁剪后需要保存的图片高度</td>
    </tr>
    <tr>
      <td align="center">isSaveRectangle</td>
      <td align="center">裁剪后的图片是按矩形区域保存还是裁剪框的形状，例如圆形裁剪的时候，该参数给true，那么保存的图片是矩形区域，如果该参数给fale，保存的图片是圆形区域</td>
    </tr>
    <tr>
      <td align="center">imageLoader</td>
      <td align="center">需要使用的图片加载器，自需要实现ImageLoader接口即可</td>
    </tr>
  </tbody>
</table>
## 3.代码参考
### 首先你要继承`com.lzy.imagepicker.loader.ImageLoader`这个接口，实现其中的方法，比如以下是使用glide第三方加载库实现的
```Java
public class GlideImageLoader implements ImageLoader {

    @Override
    public void displayImage(Activity activity, String path, ImageView imageView, int width, int height) {
        Glide.with(activity)                             //配置上下文
                .load(Uri.fromFile(new File(path)))      //设置图片路径(fix #8,文件名包含%符号 无法识别和显示)
                .error(R.mipmap.default_image)           //设置错误图片
                .placeholder(R.mipmap.default_image)     //设置占位图片
                .diskCacheStrategy(DiskCacheStrategy.ALL)//缓存全尺寸
                .into(imageView);
    }

    @Override
    public void clearMemoryCache() {
    }
}
```
### 然后配置图片选择器，一般需要在Application里初始化配置一次就可以了，将上面的图片加载器配置进去
```Java
public class MyApplication extends Application {

    @Override
    public void onCreate() {
        ImagePicker imagePicker = ImagePicker.getInstance();
        imagePicker.setImageLoader(new GlideImageLoader());   //设置图片加载器
        imagePicker.setShowCamera(true);                      //显示拍照按钮
        imagePicker.setCrop(true);                           //允许裁剪（单选才有效）
        imagePicker.setSaveRectangle(true);                   //是否按矩形区域保存
        imagePicker.setSelectLimit(9);              //选中数量限制
        imagePicker.setStyle(CropImageView.Style.CIRCLE);  //裁剪框的形状
        imagePicker.setFocusWidth(800);                       //裁剪框的宽度。单位像素（圆形自动取宽高最小值）
        imagePicker.setFocusHeight(800);                      //裁剪框的高度。单位像素（圆形自动取宽高最小值）
        imagePicker.setOutPutX(1000);                         //保存文件的宽度。单位像素
        imagePicker.setOutPutY(1000);                         //保存文件的高度。单位像素
    }
}
```
### 以上配置完成后然后在适当的地方开启相册，例如点击按钮时
```java
public void onClick(View v) {
            Intent intent = new Intent(this, ImageGridActivity.class);
            startActivityForResult(intent, IMAGE_PICKER);  
        }
    }
```
### 重写onActivityResult方法回调结果
```java
@Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == ImagePicker.RESULT_CODE_ITEMS) {
            //添加图片返回
            if (data != null && requestCode == REQUEST_CODE_SELECT) {
                ArrayList<ImageItem> images = (ArrayList<ImageItem>) data.getSerializableExtra(ImagePicker.EXTRA_RESULT_ITEMS);
                selImageList.addAll(images);
                adapter.setImages(selImageList);
            }
        } else if (resultCode == ImagePicker.RESULT_CODE_BACK) {
            //预览图片返回
            if (data != null && requestCode == REQUEST_CODE_PREVIEW) {
                ArrayList<ImageItem> images = (ArrayList<ImageItem>) data.getSerializableExtra(ImagePicker.EXTRA_IMAGE_ITEMS);
                selImageList.clear();
                selImageList.addAll(images);
                adapter.setImages(selImageList);
            }
        }
    }
```
