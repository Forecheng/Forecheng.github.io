---
title: 自定义Android中的对话框
tags:
  - Android
  - 对话框
toc: true
date: 2016-12-18 20:49:52
categories: 学习总结
---

在项目中，我们可能会经常使用到对话框，旨在告诉用户是否确定当前的操作，或者输入额外信息，对话框没有占满整个屏幕，单击对话框上的按钮或者上面的选项或者对话框外的屏幕能使对话框消失掉。关于对话框的几个类主要有`Dialog`，`AlertDialog`，其中`Dialog`是对话框的基类，`AlertDialog`是`Dialog`的子类，可用于显示最多三个按钮的对话框，也可对其进行自定义布局，此外，还有`DatePickerDialog `和 `TimePickerDialog`，可用于选择日期和时间，还有一个`ProgressDialog`，是显示进度条的对话框，常用于进行网络请求或者文件下载等。
<!--more-->

### 创建对话框的常用代码片段
```java
 AlertDialog.Builder builder = new AlertDialog.Builder(context);
        builder.setTitle(title).setMessage(message)
                .setView(view)
                .setPositiveButton(context.getText(R.string.dialog_ok_btn), new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        Log.d("dialog", "onClick: " + "click positive button");
                    }
                })
                .setNegativeButton(context.getText(R.string.dialog_cancel_btn), new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();  //取消对话框
                    }
                })
                .setNeutralButton(context.getText(R.string.dialog_neutral_btn), new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        Log.d("dialog", "onClick: " + "click neutral button");
                    }
                }).create().show();
```

<div align=center>
![运行结果](/images/dialog1.png)
</div>

对话框主要包含`标题`，`内容区域`和`按钮`。可以调用`setView(View view)`来将新的布局添加到对话框中。通过使用
` LayoutInflater ，并调用 inflate()`来得到View对象，第一个参数传入一个布局资源Id， 第二个参数是布局的父视图，一般都传入`null`。

### 自定义对话框
根据所需对话框的外观，先定义对话框的布局文件，包含标题、内容区域和按钮风格。
```java
/**
 * Description: 自定义对话框
 * Created by lpc on 2016/12/17.
 */
public class CustomDialog extends AlertDialog {


    protected CustomDialog(Context context) {
        super(context);
    }

    protected CustomDialog(Context context, int theme) {
        super(context, theme);
    }

    protected CustomDialog(Context context, boolean cancelable, OnCancelListener cancelListener) {
        super(context, cancelable, cancelListener);
    }

    public static class Builder{
        private String title;
        private String message;
        private View view;
        private String positiveButtonText;
        private String negativeButtonText;
        private String neutralButtonText;
        private DialogInterface.OnClickListener positiveOnClick;
        private DialogInterface.OnClickListener negativeOnClick;
        private DialogInterface.OnClickListener neutralOnClick;
        private Context context;

        public Builder(Context context){
            this.context = context;
        }

        public Builder setTitle(String title){
            this.title = title;
            return this;
        }

        public Builder setTitle(int resId){
            this.title = (String) context.getText(resId);
            return this;
        }

        public Builder setMessage(String message){
            this.message = message;
            return this;
        }

        public Builder setMessage(int resId){
            this.message = (String) context.getText(resId);
            return this;
        }

        public Builder setView(View view){
            this.view = view;
            return this;
        }

        public Builder setPositiveText(String text, OnClickListener listener){
            this.positiveButtonText = text;
            this.positiveOnClick = listener;
            return this;
        }
        public Builder setPositiveText(int resId, OnClickListener listener){
            this.positiveButtonText = (String) context.getText(resId);
            this.positiveOnClick = listener;
            return this;
        }

        public Builder setNegativeText(String text, OnClickListener listener){
            this.negativeButtonText = text;
            this.negativeOnClick = listener;
            return this;
        }
        public Builder setNegativeText(int resId, OnClickListener listener){
            this.negativeButtonText = (String) context.getText(resId);
            this.negativeOnClick = listener;
            return this;
        }

        public Builder setNeutralText(String text, OnClickListener listener){
            this.neutralButtonText = text;
            this.neutralOnClick = listener;
            return this;
        }
        public Builder setNeutralText(int resId, OnClickListener listener){
            this.neutralButtonText = (String) context.getText(resId);
            this.neutralOnClick = listener;
            return this;
        }

        public CustomDialog create(){
            final CustomDialog dialog = new CustomDialog(context);
            //加载自定义的对话框布局
            View dialog_view = LayoutInflater.from(context).inflate(R.layout.custom_dialog_layout, null);
            TextView textTitle = (TextView) dialog_view.findViewById(R.id.dialog_title);
            textTitle.setText(title);
            TextView textMessage = (TextView) dialog_view.findViewById(R.id.dialog_message);
            LinearLayout contentView = (LinearLayout) dialog_view.findViewById(R.id.custom_view);
            if (message != null){
                textMessage.setText(message);
            }else{
                contentView.addView(view, new ViewGroup.LayoutParams(LinearLayout.LayoutParams.MATCH_PARENT,
                        LinearLayout.LayoutParams.WRAP_CONTENT));
            }

            Button positive = (Button) dialog_view.findViewById(R.id.ok_btn);
            if (positiveButtonText != null){
                positive.setText(positiveButtonText);
                if (positiveOnClick != null){
                    positive.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            positiveOnClick.onClick(dialog,DialogInterface.BUTTON_POSITIVE);
                        }
                    });
                }
            }else{
                positive.setVisibility(View.GONE);
            }
            Button negative = (Button) dialog_view.findViewById(R.id.cancel_btn);
            if (negativeButtonText != null){
                negative.setText(negativeButtonText);
                if (negativeOnClick != null){
                    negative.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            negativeOnClick.onClick(dialog,DialogInterface.BUTTON_NEGATIVE);
                        }
                    });
                }
            }else{
                negative.setVisibility(View.GONE);
            }
            Button neutral = (Button) dialog_view.findViewById(R.id.neutral_btn);
            if (neutralButtonText != null){
                neutral.setText(neutralButtonText);
                if (neutralOnClick != null){
                    neutral.setOnClickListener(new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            neutralOnClick.onClick(dialog,DialogInterface.BUTTON_NEUTRAL);
                        }
                    });
                }
            }else{
                neutral.setVisibility(View.GONE);
            }
            dialog.setView(dialog_view);
            return dialog;
        }

    }
}
```
在其中加载自定义的对话框布局，设置对话框标题、内容文本、布局和按钮文本以及按钮行为监听。当对话框不需要显示文本内容，而是要显示一个Edittext去输入信息时，使用setView方法去加载内容布局。调用和AlertDialog一样：
```java
CustomDialog.Builder builder = new CustomDialog.Builder(context);
        builder.setTitle(title).setMessage(message).setView(view)
                .setNegativeText(R.string.dialog_cancel_btn, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                })
                .setNeutralText(R.string.dialog_neutral_btn, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        Log.d("dialog", "onClick: " + "custom --- click neutral button");
                    }
                })
                .setPositiveText(R.string.dialog_ok_btn, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        Log.d("dialog", "onClick: " + "custom --- click positive button");
                    }
                }).create().show();
```
<div align=center>
![运行结果](/images/dialog2.png)
</div>

### 使用DialogFragment
`DialogFragment`类提供创建对话框和管理其外观所需的所有控件，使用`DialogFragment`管理对话框可确保它能正确处理生命周期事件，该类是在API 11引入的，为了兼容更低的版本，应该使用支持库的中的`DialogFragment`，也就是`android.support.v4.app.DialogFragment`，通过继承该类来显示对话框。
```java
/**
 * Description:
 * Created by lpc on 2016/12/18.
 */
public class TipsDialogFragment extends DialogFragment {

    @NonNull
    @Override
    public Dialog onCreateDialog(Bundle savedInstanceState) {
        //创建对话框
        CustomDialog.Builder builder = new CustomDialog.Builder(getActivity());
        builder.setTitle(R.string.dialog_title).setMessage(R.string.dialog_message)
                .setPositiveText(R.string.dialog_ok_btn, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        //nothing
                    }
                })
                .setNegativeText(R.string.dialog_cancel_btn, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                });
        return builder.create();
    }
}
```
显示对话框：
```java
FragmentManager manager = getSupportFragmentManager();
TipsDialogFragment fragment = new TipsDialogFragment();
fragment.show(manager,"myDialog");
```
清除对话框：
除了触摸对话框让其消失外，还可以调用`DialogFragment`的`dismiss()`方法来手动清除对话框，如果在清除时要执行一些操作，则在`DialogFragment`中实现`onDismiss()`方法。如果单击“返回”按钮或者触摸对话框外部屏幕区域时，会调用`onCancel()`来取消对话框，此时系统会立即调用`onDismiss()`，因此将对话框从视图中移除时，通常应该调用`dismiss()`方法。



