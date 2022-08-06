## 1、领京豆大改版

 - 研发质量

   领京豆首页大改版基本上从零开始对领京豆首页进行重构再造，领京豆首页逻辑和交互较为复杂，但是作为日UV达六百多万的重要频道，大改版的研发质量至关重要，基于Q3领京豆首页大改版的研发需求，提出代码质量的定性指标。为了反映出代码的研发质量，特别选取了代码回滚次数、灰度进度作为代码质量的衡量指标。以8月份第一个版本为基准，对比9月份最后一个版本。

   > 1. 代码回滚率
   > 2. 灰度质量
   > 3. 投诉率

- 研发效率

  	> 1. 暂无

## 2、自研指标

​	目前负责的项目：**领京豆**、京造、关注组件、指纹组件

​	曾经参与的项目：**领现金**、东家小院、女神范、首页

```xml
- Flutter插件（接触时间短，暂无产出）
- JetPack（无项目，无法落地）
- Kotlin（无项目，无法落地）
```

 - XView支持内嵌VK、RN、Flutter控件预研

   > XView在原生层面已经支持了WebView的内嵌，但是对于VK这样的可动态下发样式的View组件缺少支持能力，VK相对于WebView有更好的用户体验。基于此缺陷，本指标将对XView支持VK样式展示的场景进行预研，计划实现XView内嵌VK控件的能力。基于实现了上述能力之后，计划进一步支持XView内嵌RN、Flutter控件的展示，具体体现为XView内嵌RN控件的能力的预研，XView内嵌Flutter控件能力的预研。自7月1日起，以9月30日为最后期限。

   实现XView内嵌VK控件的能力，并写出演示Demo，并进一步抽取出XView内嵌不同类型控件的api接口，在框架层实现XView接入不同类型内嵌控件的能力。

   

   基于实现了上述的XView内嵌不同类型控件的框架，进一步实现XView内嵌RN、Flutter控件的能力，并写出演示Demo。

   

## 3、RN可抽取组件

- 关注组件的Flutter插件；
- RN导航栏组件化，可支持沉浸式，自定义返回按钮事件，自定义分享、关注按钮等菜单栏；
- 全屏可滑动任务icon；

## 4、XViewKit

``` java
package com.jd.lib.JDCashReward.v.popup;

import android.app.Activity;
import android.content.Context;
import android.content.SharedPreferences;
import android.graphics.drawable.Drawable;
import android.support.annotation.NonNull;
import android.text.TextUtils;
import android.util.Log;
import android.view.Gravity;
import android.view.KeyEvent;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.ImageView;

import com.facebook.drawee.view.SimpleDraweeView;
import com.jd.framework.json.JDJSONObject;
import com.jd.lib.JDCashReward.R;
import com.jd.lib.JDCashReward.p.utils.CommonUtils;
import com.jd.lib.JDCashReward.p.utils.LayoutUtils;
import com.jd.viewkit.JDViewKit;
import com.jd.viewkit.helper.JDViewKitFloorModel;
import com.jd.viewkit.helper.JDViewKitViewListener;
import com.jd.viewkit.helper.JDViewKitViewListenerCallBack;
import com.jd.viewkit.helper.JDViewKitViewListenerParamsModel;
import com.jd.viewkit.templates.JDViewKitBaseLayout;
import com.jd.viewkit.thirdinterface.JDViewKitEventService;
import com.jd.viewkit.thirdinterface.JDViewKitImageService;
import com.jd.viewkit.thirdinterface.JDViewKitMtaService;
import com.jd.viewkit.thirdinterface.model.JDViewKitEventCallBack;
import com.jd.viewkit.thirdinterface.model.JDViewKitEventModel;
import com.jd.viewkit.thirdinterface.model.JDViewKitImageModel;
import com.jd.viewkit.thirdinterface.model.JDViewKitMtaModel;
import com.jd.viewkit.thirdinterface.model.JDViewKitParamsModel;
import com.jingdong.common.BaseActivity;
import com.jingdong.common.utils.CommonBase;
import com.jingdong.common.utils.JDImageUtils;
import com.jingdong.jdsdk.config.Configuration;
import com.jingdong.jdsdk.network.toolbox.HttpError;
import com.jingdong.jdsdk.network.toolbox.HttpGroup;
import com.jingdong.jdsdk.network.toolbox.HttpResponse;
import com.jingdong.jdsdk.network.toolbox.HttpSetting;
import com.jingdong.sdk.utils.DPIUtil;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;


/**
 * Created by wangshun3 on 2020/8/3
 * Des :
 */
public class XViewKit extends FrameLayout {
    private JDViewKit mJDViewKit;
    private JDViewKitFloorModel mJDViewKitFloorModel;
    private ViewGroup mXViewKitContainer;
    private JDViewKitBaseLayout jdViewKitBaseLayout;
    private static final String SHOW_XVIEWKIT_COUNT_DAILY = "SHOW_XVIEWKIT_COUNT_DAILY22";
    private static XViewKit mXViewKit;
    private int showCount = Integer.MAX_VALUE;
    private boolean isInterceptBackKey;

    private XViewKit(@NonNull Context context) {
        super(context);
    }

    public static XViewKit getInstance(Context context) {
        if (mXViewKit == null) {
            synchronized (XViewKit.class) {
                if (mXViewKit == null) {
                    mXViewKit = new XViewKit(context);
                }
            }
        }
        return mXViewKit;
    }

    public void showView() {

        if (showCount != Integer.MAX_VALUE && showCount <= getXViewKitShowCountDaily()) {
            Log.d("XViewKit", "showView return null " + showCount + " - " + getXViewKitShowCountDaily());
            return;
        }

        if (getContext() instanceof Activity) {
            Activity activity = (Activity) getContext();
            View decorView = activity.getWindow().getDecorView();
            if (decorView instanceof ViewGroup) {
                mXViewKitContainer = (ViewGroup) decorView;
            }
        }
        if (mXViewKitContainer != null) {
            mXViewKitContainer.removeView(XViewKit.this);
            LayoutParams layoutParams = new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
            mXViewKitContainer.addView(XViewKit.this, layoutParams);

            LayoutParams XViewLp = new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
            XViewKit.this.setBackgroundColor(CommonUtils.getAlphaColor("#000000", 0.9f));
            XViewKit.this.setLayoutParams(XViewLp);

            XViewKit.this.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {

                }
            });
        }
        requestXViewKitData();
    }

    private void addCloseView() {
        ImageView imgClose = new ImageView(getContext());
        LayoutUtils.addViewWithWidthHeightAndMarginsByDpi750(XViewKit.this, imgClose, 60, 60, 630, 120);
        imgClose.setBackgroundResource(R.drawable.new_close);
        imgClose.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                closeXViewKit();
            }
        });
    }

    private void requestXViewKitData() {
        final BaseActivity activity = (BaseActivity) getContext();
        HttpSetting httpSetting = new HttpSetting();
        httpSetting.setFunctionId("queryVkComponent");
        httpSetting.setHost(Configuration.getPortalHost());
        httpSetting.setOnTouchEvent(true);
        httpSetting.setUseFastJsonParser(true);
        httpSetting.setListener(new HttpGroup.OnAllListener() {
            @Override
            public void onStart() {
            }

            @Override
            public void onProgress(int i, int i1) {
            }

            @Override
            public void onError(HttpError httpError) {
            }

            @Override
            public void onEnd(HttpResponse httpResponse) {
                JDJSONObject rootObject = httpResponse.getFastJsonObject();
                initViewKit(rootObject.optString("dslMap"));
                JDJSONObject componentData = rootObject.optJSONObject("componentData");
                if (componentData != null) {
                    mJDViewKitFloorModel = mJDViewKit.getFollrModelByDsl(componentData.optString("dslRoot"), componentData.optString("dslData"));
                    updateView();
                }
            }
        });
        activity.getHttpGroupaAsynPool().add(httpSetting);
    }

    private void updateView() {
        post(new Runnable() {
            @Override
            public void run() {
                try {
                    if (mJDViewKitFloorModel == null) {
                        return;
                    }
                    XViewKit.this.removeAllViews();
                    jdViewKitBaseLayout = mJDViewKit.getRootLayoutByFloorId(getContext(), mJDViewKitFloorModel.getFloorId());
                    jdViewKitBaseLayout.setFloorModel(mJDViewKitFloorModel);
                    jdViewKitBaseLayout.isShow(true);
                    XViewKit.this.addView(jdViewKitBaseLayout);

                    LayoutParams viewKitLp = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
                    viewKitLp.gravity = Gravity.CENTER;
                    jdViewKitBaseLayout.setLayoutParams(viewKitLp);

                    addCloseView();

                    if (showCount != Integer.MAX_VALUE) {
                        saveXViewKitShowCountDaily();
                    }

                } catch (Exception e) {
                    Log.d("XViewKit", "XViewKit - " + "updateView() called " + e.getMessage());
                }
            }
        });

        if (mJDViewKitFloorModel != null) {
            mJDViewKitFloorModel.setViewListener(new JDViewKitViewListener() {
                @Override
                public void floorRefresh(JDViewKitViewListenerParamsModel jdViewKitViewListenerParamsModel, JDViewKitViewListenerCallBack jdViewKitViewListenerCallBack) {
                    Log.d("XViewKit", "XViewKit - " + "floorRefresh() called ");
                }
            });
        }
    }

    private void initViewKit(String dslMap) {
        try {
            if (mJDViewKit == null) {
//                mJDViewKit = new JDViewKit(DPIUtil.getWidth(getContext()), new JDVKitFloatEventServiceImpl(),
//                        JDVKitImageServiceImpl.getInstance(), JDVKitMtaServiceImpl.getInstance(),
//                        new JDViewKitParamsModel("", ""));


                mJDViewKit = new JDViewKit(DPIUtil.getAppWidth((Activity) getContext()), new JDViewKitEventService() {
                    @Override
                    public void clickEvent(JDViewKitEventModel jdViewKitEventModel, Context context) {

                    }

                    @Override
                    public void clickEvent(JDViewKitEventModel jdViewKitEventModel, Context context, JDViewKitEventCallBack jdViewKitEventCallBack) {
                    }

                    @Override
                    public String getI18nString() {
                        return null;
                    }

                    @Override
                    public boolean isLogin() {
                        return false;
                    }
                }, new JDViewKitImageService() {
                    @Override
                    public void fillImageView(ImageView imageView, JDViewKitImageModel imageModel) {
                        if (imageView != null && imageModel != null) {
                            imageView.setScaleType(imageModel.getScaleType());
                            if (imageView instanceof SimpleDraweeView) {
                                SimpleDraweeView simpleDraweeView = (SimpleDraweeView) imageView;
                                JDImageUtils.displayImage(imageModel.getImageUrlStr(), simpleDraweeView);
                            }
                        }
                    }

                    @Override
                    public ImageView getThirdImageView(Context context) {
                        return new SimpleDraweeView(context);
                    }

                    @Override
                    public void downloadImage(Context context, String s, int i, int i1, DownloadImageListener downloadImageListener) {

                    }

                    @Override
                    public Drawable getPlaceholderDrawable(Context context, int i) {
                        return null;
                    }
                }, new JDViewKitMtaService() {
                    @Override
                    public void click(JDViewKitMtaModel jdViewKitMtaModel, Context context) {

                    }

                    @Override
                    public void pageLevel(JDViewKitMtaModel jdViewKitMtaModel, Context context) {

                    }

                    @Override
                    public void pv(JDViewKitMtaModel jdViewKitMtaModel, Context context) {

                    }
                }, new JDViewKitParamsModel("", ""));
            }
            mJDViewKit.setDslMapByStr(dslMap);
        } catch (Throwable throwable) {
            Log.e("XViewKit", "XViewKit - " + "initViewKit() called " + throwable.getMessage());
        }
    }

    public void closeXViewKit() {
        if (jdViewKitBaseLayout != null) {
            jdViewKitBaseLayout.isShow(false);
        }
        if (mXViewKitContainer != null) {
            mXViewKitContainer.removeView(XViewKit.this);
        }
    }

    public XViewKit setXViewKitShowCountDaily(int count) {
        this.showCount = count;
        return this;
    }

    private void saveXViewKitShowCountDaily() {
        SharedPreferences sp = CommonBase.getJdSharedPreferences();
        if (sp != null) {
            int countDaily = getXViewKitShowCountDaily();
            ++countDaily;

            Log.d("XViewKit", "XViewKit: " + getTimeStamp() + "_" + countDaily);

            sp.edit().putString(SHOW_XVIEWKIT_COUNT_DAILY, getTimeStamp() + "_" + countDaily).apply();
        }
    }

    private int getXViewKitShowCountDaily() {
        SharedPreferences sp = CommonBase.getJdSharedPreferences();
        if (sp != null) {
            String showCountStr = sp.getString(SHOW_XVIEWKIT_COUNT_DAILY, null);
            return getShowCountFromStr(showCountStr);
        }
        return 0;
    }

    private int getShowCountFromStr(String showCountStr) {
        if (!TextUtils.isEmpty(showCountStr) && showCountStr.contains("_")) {
            String[] split = showCountStr.split("_");
            if (split.length == 2) {
                String timeStamp = split[0];
                if (!TextUtils.isEmpty(timeStamp) && timeStamp.equals(getTimeStamp())) {
                    if (!TextUtils.isEmpty(split[1]) && TextUtils.isDigitsOnly(split[1])) {
                        return Integer.parseInt(split[1]);
                    }
                }
            }
        }
        return 0;
    }

    private String getTimeStamp() {
        try {
            SimpleDateFormat fmt = new SimpleDateFormat("yyyy-MM-dd", Locale.getDefault());
            return fmt.format(new Date());
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public XViewKit setInterceptBackKey(boolean isInterceptBackKey) {
        this.isInterceptBackKey = isInterceptBackKey;
        return this;
    }

    public boolean getInterceptBackKey() {
        return this.isInterceptBackKey;
    }

    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        this.setFocusableInTouchMode(true);
        this.requestFocus();
        this.setOnKeyListener(new OnKeyListener() {
            @Override
            public boolean onKey(View v, int keyCode, KeyEvent event) {
                Log.d("XViewKit", "XViewKit: onKey " + getInterceptBackKey());
                if (getInterceptBackKey() && event.getAction() == KeyEvent.ACTION_DOWN && keyCode == KeyEvent.KEYCODE_BACK) {
                    showView();
                    return true;
                }
                return false;
            }
        });
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        if (jdViewKitBaseLayout != null) {
            jdViewKitBaseLayout.isShow(false);
            jdViewKitBaseLayout = null;
        }
        mJDViewKitFloorModel = null;
        mJDViewKit = null;
        mXViewKit = null;
    }
}

```

## 5、XRNView

```java
package com.jd.lib.JDCashReward.v.popup;

import android.animation.ObjectAnimator;
import android.app.Activity;
import android.content.Context;
import android.graphics.Color;
import android.text.TextUtils;
import android.view.Gravity;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.ImageView;

import com.facebook.react.BuildConfig;
import com.facebook.react.ReactInstanceManager;
import com.facebook.react.ReactPackage;
import com.facebook.react.ReactRootView;
import com.facebook.react.common.LifecycleState;
import com.facebook.react.shell.MainReactPackage;
import com.jd.lib.JDCashReward.R;
import com.jingdong.common.babelrn.view.BabelRNFloor;
import com.jingdong.common.jdreactFramework.activities.JDReactRootView;
import com.jingdong.common.jdreactFramework.download.PluginVersion;
import com.jingdong.common.jdreactFramework.download.ReactNativeFileManager;
import com.jingdong.common.jdreactFramework.preload.JDReactPreloadManager;
import com.jingdong.common.jdreactFramework.utils.AbstractJDReactInitialHelper;
import com.jingdong.common.jdreactFramework.utils.ReactModuleAvailabilityUtils;
import com.jingdong.common.jdreactFramework.utils.ReactVersionUtils;
import com.jingdong.corelib.utils.Log;
import com.jingdong.jdsdk.JdSdk;
import com.jingdong.sdk.utils.DPIUtil;

import java.io.File;
import java.lang.reflect.Field;

/**
 * Created by wangshun3 on 2020/8/11
 * Des :
 */
public class XRNView extends FrameLayout {
    public static final String TAG = XRNView.class.getSimpleName();
    private static XRNView mXRNView;
    private static ReactInstanceManager mReactInstanceManager;
    private ViewGroup mDecorView;
    private JDReactRootView mReactRootView;
    private String bundlePath;
    private ImageView imgClose;
    private String bundleName = "JDReactCollectJDBeans";

    private XRNView(Context context) {
        super(context);
    }

    public static XRNView getInstance(Context context) {
        if (mXRNView == null) {
            synchronized (XRNView.class) {
                if (mXRNView == null) {
                    mXRNView = new XRNView(context);
                }
            }
        }
        return mXRNView;
    }

    public void showXRNView() {
        if (getContext() instanceof Activity) {
            Activity activity = (Activity) getContext();
            View decorView = activity.getWindow().getDecorView();
            if (decorView instanceof ViewGroup) {
                mDecorView = (ViewGroup) decorView;
            }
        }

        if (mDecorView != null) {
            mXRNView.removeAllViews();
            mDecorView.removeView(mXRNView);
            LayoutParams layoutParams = new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
            mDecorView.addView(mXRNView, layoutParams);

            LayoutParams XViewLp = new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
            mXRNView.setBackgroundColor(getAlphaColor("#000000", 0.9f));
            mXRNView.setLayoutParams(XViewLp);

            mXRNView.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {

                }
            });

            initRNLayout();
        }
    }

    private void initRNLayout() {
//        mReactRootView = new ReactRootView(getContext());
//        mReactRootView.setVisibility(VISIBLE);
        PluginVersion pluginAbstractInfo = ReactNativeFileManager.getPluginDir(getContext(), "JDReactCollectJDBeans");
        if (pluginAbstractInfo != null && pluginAbstractInfo.pluginDir != null) {
            bundlePath = pluginAbstractInfo.pluginDir + File.separator + "JDReactCollectJDBeans.jsbundle";
            Log.e(TAG, " bundlePath" + bundlePath);
        }

        mReactRootView = new JDReactRootView((Activity) getContext(), "JDReactCollectJDBeans", "JDReactCollectJDBeans", null);

        LayoutParams rnViewLp = new FrameLayout.LayoutParams(DPIUtil.getAppWidth((Activity) getContext()) * 2 / 3, DPIUtil.getAppHeight((Activity) getContext()) / 2);
        rnViewLp.gravity = Gravity.CENTER;
        mReactRootView.setLayoutParams(rnViewLp);
        mXRNView.addView(mReactRootView);


        addCloseView();
    }

    public ReactInstanceManager getReactInstanceManager1() {

        boolean reactModuleAvail = ReactModuleAvailabilityUtils.getModuleAvailability("JDReactCollectJDBeans");
        if (!reactModuleAvail) {
            System.out.println(TAG + " reactModuleAvail");
            return null;
        }

        AbstractJDReactInitialHelper mAbstractJDReactInitialHelper = new AbstractJDReactInitialHelper((Activity) getContext(), bundleName, "", bundleName, null, bundlePath, null) {
            @Override
            protected void initRootView(ReactRootView rootView) {
                System.out.println(TAG + " rootView: " + rootView);
            }

            @Override
            protected void defaultOnBackPressed() {
            }

            @Override
            protected ReactPackage getDefaultReactPackage() {
                return AbstractJDReactInitialHelper.getPackageManger();
            }

            @Override
            protected ReactPackage getExtendReactPackage() {
                return null;
            }
        };
        mAbstractJDReactInitialHelper.initReactManager(null, bundlePath, "", (Activity) getContext(), bundleName, bundleName, null, false, 0, "");

        return mAbstractJDReactInitialHelper.getReactManager();
    }


    private boolean isInit(ReactRootView reactRootView) {
        if (null != reactRootView) {
            try {
                Field field = ReactRootView.class.getDeclaredField("mReactInstanceManager");
                field.setAccessible(true);
                return field.get(reactRootView) != null;
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return false;
    }

    public ReactInstanceManager getReactInstanceManager() {
        if (mReactInstanceManager == null) {
            // If the bundle exists in app internal directory
            if (!TextUtils.isEmpty(bundlePath)) {
                int result = ReactVersionUtils.weUsePreloadData(JdSdk.getInstance().getApplication(), "JDReactCollectJDBeans");
                if (bundlePath.startsWith("/data/data/")) {
                    result = 0;
                }
                if (result == 1 || result == -1) {
                    mReactInstanceManager = ReactInstanceManager.builder()
                            .setApplication(JdSdk.getInstance().getApplication())
//                        .setSeperateBundleAssetName(bundlePath, "jdreact/JDReactCommon/JDReactCommon.jsbundle")
                            .setSeperateBundleAssetName(bundlePath, "jdreact/JDReactCollectJDBeans/JDReactCollectJDBeans.jsbundle")
                            .setJSMainModulePath("jsbundles/JDReactCollectJDBeans")
                            .addPackage(new MainReactPackage())
//                            .setSetupReactContextInBackgroundEnabled(true)
                            .setUseDeveloperSupport(BuildConfig.DEBUG)
                            .setInitialLifecycleState(LifecycleState.BEFORE_CREATE)
                            .build();
                } else {
                    mReactInstanceManager = ReactInstanceManager.builder()
                            .setApplication(JdSdk.getInstance().getApplication())
                            .setJSBundleFile(bundlePath)
                            .setJSMainModulePath("jsbundles/JDReactCollectJDBeans")
                            .addPackage(new MainReactPackage())
                            // .setSetupReactContextInBackgroundEnabled(true)
                            .setInitialLifecycleState(LifecycleState.BEFORE_RESUME)
                            .build();
                }
            }
        }
        return mReactInstanceManager;
    }

    private void getRNView(String reactBundle, String bundlePath) {
        System.out.println(TAG + " reactBundle:" + reactBundle + " bundlePath: " + bundlePath);
        JDReactPreloadManager jdReactPreloadManager = JDReactPreloadManager.getPreloadManagerByModule(reactBundle);
        if (!TextUtils.isEmpty(bundlePath) && jdReactPreloadManager != null && bundlePath.equals(jdReactPreloadManager.getBundlePath())) {
//            mReactRootView = jdReactPreloadManager.getReactRootView();
        }
    }

    private void addCloseView() {
        imgClose = new ImageView(getContext());
        FrameLayout.LayoutParams imgCloseLp = new FrameLayout.LayoutParams(FrameLayout.LayoutParams.WRAP_CONTENT, FrameLayout.LayoutParams.WRAP_CONTENT);
        imgCloseLp.width = 80;
        imgCloseLp.height = 80;
        imgCloseLp.rightMargin = 150;
        imgCloseLp.topMargin = 150;
        imgCloseLp.gravity = Gravity.RIGHT;
        imgClose.setLayoutParams(imgCloseLp);
        imgClose.setBackgroundResource(R.drawable.new_close);

        mDecorView.addView(imgClose);

        imgClose.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                closeXRNView();
            }
        });
    }

    private void closeXRNView() {
        Log.e(TAG, "closeXRNView");
        if (mDecorView != null) {
            mDecorView.removeView(imgClose);
            mDecorView.removeView(mXRNView);
            mReactInstanceManager = null;
            mXRNView = null;
        }
    }

    public static int getAlphaColor(String colorStr, float alpha) {
        if (colorStr.length() == 7) {
            String[] split = colorStr.split("#");
            String alphaStr = Integer.toHexString((int) (alpha * 255));
            return Color.parseColor("#" + alphaStr + split[1]);
        }
        return Color.BLACK;
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        Log.e(TAG, "onDetachedFromWindow");

        mDecorView = null;
        mReactRootView = null;
        mReactInstanceManager = null;
        mXRNView = null;
    }
}
```

> 精简版本：
>
> 1. 使用JDReactRootView作为弹窗的content，内部封装关闭按钮事件等，更多事件的处理待处理。

 ``` java
package com.jd.lib.JDCashReward.v.popup;

import android.app.Activity;
import android.content.Context;
import android.graphics.Color;
import android.text.TextUtils;
import android.view.Gravity;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.ImageView;

import com.jd.lib.JDCashReward.R;
import com.jingdong.common.jdreactFramework.activities.JDReactRootView;
import com.jingdong.corelib.utils.Log;
import com.jingdong.sdk.utils.DPIUtil;

/**
 * Created by wangshun3 on 2020/8/11
 * Des :
 */
public class XRNView extends FrameLayout {
    public static final String TAG = XRNView.class.getSimpleName();
    private static XRNView mXRNView;
    private ViewGroup mDecorView;
    private JDReactRootView mReactRootView;
    private ImageView imgClose;

    private XRNView(Context context) {
        super(context);
    }

    public static XRNView getInstance(Context context) {
        if (mXRNView == null) {
            synchronized (XRNView.class) {
                if (mXRNView == null) {
                    mXRNView = new XRNView(context);
                }
            }
        }
        return mXRNView;
    }

    public void showXRNView(String bundleName) {

        if (TextUtils.isEmpty(bundleName)) {
            return;
        }

        if (getContext() instanceof Activity) {
            Activity activity = (Activity) getContext();
            View decorView = activity.getWindow().getDecorView();
            if (decorView instanceof ViewGroup) {
                mDecorView = (ViewGroup) decorView;
            }
        }

        if (mDecorView != null) {
            mXRNView.removeAllViews();
            mDecorView.removeView(mXRNView);
            LayoutParams layoutParams = new LayoutParams(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
            mDecorView.addView(mXRNView, layoutParams);

            LayoutParams XViewLp = new LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
            mXRNView.setBackgroundColor(getAlphaColor("#000000", 0.9f));
            mXRNView.setLayoutParams(XViewLp);

            mXRNView.setOnClickListener(new OnClickListener() {
                @Override
                public void onClick(View v) {

                }
            });

            initRNLayout(bundleName);
        }
    }

    private void initRNLayout(String bundleName) {
        mReactRootView = new JDReactRootView((Activity) getContext(), bundleName, bundleName, null);

        LayoutParams rnViewLp = new FrameLayout.LayoutParams(DPIUtil.getWidthByDesignValue750(getContext(), 700), DPIUtil.getAppHeight((Activity) getContext()) / 2);
        rnViewLp.gravity = Gravity.CENTER;
        mReactRootView.setLayoutParams(rnViewLp);
        mXRNView.addView(mReactRootView);

        addCloseView();
    }

    private void addCloseView() {
        imgClose = new ImageView(getContext());
        FrameLayout.LayoutParams imgCloseLp = new FrameLayout.LayoutParams(FrameLayout.LayoutParams.WRAP_CONTENT, FrameLayout.LayoutParams.WRAP_CONTENT);
        imgCloseLp.width = 80;
        imgCloseLp.height = 80;
        imgCloseLp.rightMargin = 150;
        imgCloseLp.topMargin = 150;
        imgCloseLp.gravity = Gravity.RIGHT;
        imgClose.setLayoutParams(imgCloseLp);
        imgClose.setBackgroundResource(R.drawable.new_close);

        mDecorView.addView(imgClose);

        imgClose.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                closeXRNView();
            }
        });
    }

    private void closeXRNView() {
        Log.e(TAG, "closeXRNView");
        if (mDecorView != null) {
            mDecorView.removeView(imgClose);
            mDecorView.removeView(mXRNView);
            mReactRootView = null;
            mDecorView=null;
            mXRNView = null;
        }
    }

    public static int getAlphaColor(String colorStr, float alpha) {
        if (colorStr.length() == 7) {
            String[] split = colorStr.split("#");
            String alphaStr = Integer.toHexString((int) (alpha * 255));
            return Color.parseColor("#" + alphaStr + split[1]);
        }
        return Color.BLACK;
    }

    @Override
    protected void onDetachedFromWindow() {
        super.onDetachedFromWindow();
        Log.e(TAG, "onDetachedFromWindow");
        if (mDecorView != null) {
            mReactRootView = null;
            mDecorView=null;
            mXRNView = null;
        }
    }
}
 ```


