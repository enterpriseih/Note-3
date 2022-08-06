# bindX-RN

## RN 

```react
TimingDemo.js
import {
    StyleSheet,
    View,
    Text,
    NativeModules,
    findNodeHandle,
    TouchableHighlight,
    ToastAndroid,
    PanResponder
} from 'react-native';
onBind(){
        let anchor = findNodeHandle(this.refs._anchor);
        let token = bindingx.bind({
            eventType: 'orientation',
            options: {
                sceneType: '2d' //2d场景会返回x,y分量
            },
            props: [
                {
                    element: anchor,
                    property: 'transform.translateX',
                    expression: 'x+0'
                },
                {
                    element: anchor,
                    property: 'transform.translateY',
                    expression: 'y+0'
                }
            ]
        });

        this._token = token.token;
    }

<View ref="_anchor" style={{
                      width: 100,
                      height: 100,
                      backgroundColor: '#00ff00',
                      justifyContent: 'center',
                      alignItems: 'center'
                    }}>
                        <Text style={styles.text}>Target</Text>

                    </View>
```

```react
bindingx.js
bind(options, callback = function () {
  }) {
    if (!options) {
      throw new Error('should pass options for binding');
    }
    options.exitExpression = formatExpression(options.exitExpression);
    console.log('options:',options)
    if (options.props) {
      options.props.forEach((prop) => {
        prop.expression = formatExpression(prop.expression);
      });
    }
    let res;

    if (nativeBindingX) {

      res = nativeBindingX.bind(options);
      let token = res && res.token;
      this.__instances__[token] = {
        callback
      };
    }
    return res;
  }
```

```java
ReactBindingXModule.java
public WritableMap bind(final ReadableMap params) {
        final CountDownLatch latch = new CountDownLatch(1);
        final List<String> resultHolder = new ArrayList<>(2);
        executeAsynchronously(new Runnable() {
            @Override
            public void run() {
                try {
                    prepareInternal();
                    String token = mBindingXCore.doBind(
                            getReactApplicationContext(),
                            null,// react native don't need it
                            params == null ? Collections.<String, Object>emptyMap() : params.toHashMap(),
                            new BindingXCore.JavaScriptCallback() {
                                @Override
                                public void callback(Object params) {
                                    ReactApplicationContext context = getReactApplicationContext();
                                    if(context != null) {
                                        context.getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
                                                .emit("bindingx:statechange",Arguments.makeNativeMap((Map<String,Object>)params));
                                    }
                                }
                            });
                    resultHolder.add(token);
                }finally {
                    latch.countDown();
                }
            }
        });
        try {
            latch.await(2000, TimeUnit.MILLISECONDS);
        }catch (Exception e) {
            //ignore
        }

        String token = resultHolder.size() > 0 ? resultHolder.get(0) : null;
        return Arguments.makeNativeMap(Collections.<String,Object>singletonMap(BindingXConstants.KEY_TOKEN, token));
    }
```

```java
BindingXCore.java
  
public String doBind(@Nullable Context context,
                         @Nullable String instanceId,
                         @NonNull Map<String, Object> params,
                         @NonNull JavaScriptCallback callback, Object... extension) {
        String eventType = Utils.getStringValue(params, BindingXConstants.KEY_EVENT_TYPE);
        String anchorInstanceId = Utils.getStringValue(params, BindingXConstants.KEY_INSTANCE_ID);
        LogProxy.enableLogIfNeeded(params);
        Object configObj = params.get(BindingXConstants.KEY_OPTIONS);
        Map<String, Object> configMap = null;
        if(configObj != null && configObj instanceof Map) {
            try {
                configMap = Utils.toMap(new JSONObject((Map)configObj));
            } catch (Exception e) {
                LogProxy.e("parse external config failed.\n", e);
            }
        }

        ExpressionPair exitExpressionPair = Utils.getExpressionPair(params, BindingXConstants.KEY_EXIT_EXPRESSION);

        String anchor = Utils.getStringValue(params, BindingXConstants.KEY_ANCHOR); // maybe nullable
        List<Map<String, Object>> expressionArgs = Utils.getRuntimeProps(params);
        Map<String,ExpressionPair> interceptors = Utils.getCustomInterceptors(params);
        return doBind(anchor, anchorInstanceId, eventType, configMap, exitExpressionPair, expressionArgs, interceptors, callback, context, instanceId, extension);
    }
```

```java
BindingXOrientationHandler.java
->AbstractEventHandler.java
 @Override
    public void onBindExpression(@NonNull String eventType,
                                 @Nullable Map<String,Object> globalConfig,
                                 @Nullable ExpressionPair exitExpressionPair,
                                 @NonNull List<Map<String, Object>> expressionArgs,
                                 @Nullable BindingXCore.JavaScriptCallback callback) {
        clearExpressions();
        transformArgs(eventType, expressionArgs);
        this.mCallback = callback;
        this.mExitExpressionPair = exitExpressionPair;

        if(!mScope.isEmpty()) {
            mScope.clear();
        }
        applyFunctionsToScope();
    }

private void transformArgs(@NonNull String eventType, @NonNull List<Map<String, Object>> originalArgs) {
        if (mExpressionHoldersMap == null) {
            mExpressionHoldersMap = new HashMap<>();
        }
        for (Map<String, Object> arg : originalArgs) {
            String targetRef = Utils.getStringValue(arg, BindingXConstants.KEY_ELEMENT);
            String targetInstanceId = Utils.getStringValue(arg, BindingXConstants.KEY_INSTANCE_ID);
            String property = Utils.getStringValue(arg, BindingXConstants.KEY_PROPERTY);

            ExpressionPair expressionPair = Utils.getExpressionPair(arg, BindingXConstants.KEY_EXPRESSION);

            Object configObj = arg.get(BindingXConstants.KEY_CONFIG);
            Map<String,Object> configMap = null;
            if(configObj != null && configObj instanceof Map) {
                try {
                    configMap = Utils.toMap(new JSONObject((Map) configObj));
                }catch (Exception e) {
                    LogProxy.e("parse config failed", e);
                }
            }

            if (TextUtils.isEmpty(targetRef) || TextUtils.isEmpty(property) || expressionPair == null) {
                LogProxy.e("skip illegal binding args[" + targetRef + "," + property + "," + expressionPair + "]");
                continue;
            }
            ExpressionHolder holder = new ExpressionHolder(targetRef,targetInstanceId, expressionPair, property, eventType, configMap);

            List<ExpressionHolder> holders = mExpressionHoldersMap.get(targetRef);
            if (holders == null) {
                holders = new ArrayList<>(4);
                mExpressionHoldersMap.put(targetRef, holders);
                holders.add(holder);
            } else if (!holders.contains(holder)) {
                holders.add(holder);
            }
        }
    }

protected void consumeExpression(@Nullable Map<String, List<ExpressionHolder>> args, @NonNull Map<String,Object> scope,
                           @NonNull String currentType) throws IllegalArgumentException, JSONException {
        tryInterceptAllIfNeeded(scope);

        if (args == null) {
            LogProxy.e("expression args is null");
            return;
        }
        if (args.isEmpty()) {
            LogProxy.e("no expression need consumed");
            return;
        }

        if(LogProxy.sEnableLog) {
            LogProxy.d(String.format(Locale.getDefault(), "consume expression with %d tasks. event type is %s",args.size(),currentType));
        }
        List<Object> extension = new LinkedList<>();
        for (List<ExpressionHolder> holderList : args.values()) {
            for (ExpressionHolder holder : holderList) {
                if (!currentType.equals(holder.eventType)) {
                    LogProxy.d("skip expression with wrong event type.[expected:" + currentType + ",found:" + holder.eventType + "]");
                    continue;
                }
                extension.clear();
                if(mExtensionParams != null && mExtensionParams.length > 0) {
                    Collections.addAll(extension, mExtensionParams);
                }

                String instanceId = TextUtils.isEmpty(holder.targetInstanceId)? mInstanceId : holder.targetInstanceId;
                if(!TextUtils.isEmpty(instanceId)) {
                    extension.add(instanceId);
                }

                ExpressionPair expressionPair = holder.expressionPair;
                if(!ExpressionPair.isValid(expressionPair)) {
                    continue;
                }
                Expression expression = mCachedExpressionMap.get(expressionPair.transformed);
                if (expression == null) {
                    expression = new Expression(expressionPair.transformed);
                    mCachedExpressionMap.put(expressionPair.transformed, expression);
                }

                Object obj = expression.execute(scope);
                if (obj == null) {
                    LogProxy.e("failed to execute expression,expression result is null");
                    continue;
                }
                if((obj instanceof Double) && Double.isNaN((Double) obj) ||
                        (obj instanceof Float && Float.isNaN((Float)obj))) {
                    LogProxy.e("failed to execute expression,expression result is NaN");
                    continue;
                }
                //apply transformation/layout change ... to target view.

                View targetView = mPlatformManager.getViewFinder().findViewBy(holder.targetRef, extension.toArray());
                BindingXPropertyInterceptor.getInstance().performIntercept(
                        targetView,
                        holder.prop,
                        obj,
                        mPlatformManager.getResolutionTranslator(),
                        holder.config,
                        holder.targetRef,
                        instanceId
                );

                if (targetView == null) {
                    LogProxy.e("failed to execute expression,target view not found.[ref:" + holder.targetRef + "]");
                    continue;
                }

                // default behavior
                mPlatformManager.getViewUpdater().synchronouslyUpdateViewOnUIThread(
                        targetView,
                        holder.prop,
                        obj,
                        mPlatformManager.getResolutionTranslator(),
                        holder.config,
                        holder.targetRef,/*additional params for weex*/
                        instanceId       /*additional params for weex*/
                );
            }
        }

    }
```

## Native

### Mock json data

```json
String timingConfig = :
{
  "eventType": "timing",
  "exitExpression": {
    "origin": "t>2000",
    "transformed": {
      "type": ">",
      "children": [
        {
          "type": "Identifier",
          "value": "t"
        },
        {
          "type": "NumericLiteral",
          "value": 2000
        }
      ]
    }
  },
  "props": [
    {
      "element": "target",
      "property": "transform.translateX",
      "expression": {
        "origin": "linear(t,0,400,2000)",
        "transformed": {
          "type": "CallExpression",
          "children": [
            {
              "type": "Identifier",
              "value": "linear"
            },
            {
              "type": "Arguments",
              "children": [
                {
                  "type": "Identifier",
                  "value": "t"
                },
                {
                  "type": "NumericLiteral",
                  "value": 0
                },
                {
                  "type": "NumericLiteral",
                  "value": 400
                },
                {
                  "type": "NumericLiteral",
                  "value": 2000
                }
              ]
            }
          ]
        }
      }
    },
    {
      "element": "target",
      "property": "background-color",
      "expression": {
        "origin": "evaluateColor('#ff0000','#607D8B',min(t,2000)/2000)",
        "transformed": {
          "type": "CallExpression",
          "children": [
            {
              "type": "Identifier",
              "value": "evaluateColor"
            },
            {
              "type": "Arguments",
              "children": [
                {
                  "type": "StringLiteral",
                  "value": "'#ff0000'"
                },
                {
                  "type": "StringLiteral",
                  "value": "'#607D8B'"
                },
                {
                  "type": "/",
                  "children": [
                    {
                      "type": "CallExpression",
                      "children": [
                        {
                          "type": "Identifier",
                          "value": "min"
                        },
                        {
                          "type": "Arguments",
                          "children": [
                            {
                              "type": "Identifier",
                              "value": "t"
                            },
                            {
                              "type": "NumericLiteral",
                              "value": 2000
                            }
                          ]
                        }
                      ]
                    },
                    {
                      "type": "NumericLiteral",
                      "value": 2000
                    }
                  ]
                }
              ]
            }
          ]
        }
      }
    }
  ]
}
```

### 源码调用链路符

> #：类内部调用
>
> @: 响应或者回调
>
> []: 调用分解
>
> {}: 部分重要代码
>
> ->: 跨类调用
>
> =>: 等价调用 直接return的方法调用
>
> ==>: 类比
>
> -->：调用太长，另起一段
>
> ~>: 暂未定
>
> ^:调用父类方法
>
> v: 调用实现类或者子类中的方法
>
> //: 注释
>
> :: 示例
>
> []:可选内容

### 源码分析

#### 动画执行的源码调用链路

> 此调用链路来源于生啃代码，未经过断点验证。

```java
Demo1Activity#bindTiming(){
  mNativeBindingX.bind(mRootView, JSON.parseObject(timingConfig), new NativeCallback() {
            @Override
            public void callback(Map<String, Object> params) {
                Log.v("CHUYI",params.toString());
            }
        });
}
->NativeBindingX#bind(View rootView, BindingXSpec spec, NativeCallback callback)
#bind(View rootView, Map<String,Object> params,final NativeCallback callback){
  mBindingXCore.doBind(rootView.getContext(), null, params, new BindingXCore.JavaScriptCallback() {
                    @Override
                    public void callback(Object params) {
                        if (callback != null && params instanceof Map) {
                            callback.callback((Map<String, Object>) params);
                        }
                    }
                },rootView);
}
->BindingXCore#doBind(...)
=>doBind(...){
  doPrepare(context, instanceId, anchor, anchorInstanceId, eventType, globalConfig){
  	//通过eventType创建EventHandler
	}
	handler.onBindExpression(eventType, globalConfig, exitExpressionPair, expressionArgs, callback)
}
->BindingXTimingHandler [ extends AbstractEventHandler implements AnimationFrame.Callback{} ]
  #onBindExpression(...){//在BindingXCore构造函数中进行registerEventHandler
  ^ AbstractEventHandler#onBindExpression(...){
  	transformArgs(eventType, expressionArgs);//动画参数解析
	}
  mAnimationFrame.requestAnimationFrame(this);
}
->AnimationFrame$HandlerAnimationFrameImpl#requestAnimationFrame(@NonNull Callback callback){
  mInnerHandler.sendEmptyMessage(MSG_FRAME_CALLBACK);
}
@handleMessage(Message msg){
  callback.doFrame();
}
v BindingXTimingHandler#doFrame()
#handleTimingCallback(){
  ^ AbstractEventHandler#consumeExpression(mExpressionHoldersMap, mScope, BindingXEventType.TYPE_TIMING){
      mPlatformManager.getViewUpdater().synchronouslyUpdateViewOnUIThread(
                            targetView,
                            holder.prop,
                            obj,
                            mPlatformManager.getResolutionTranslator(),
                            holder.config,
                            holder.targetRef,
                            instanceId
                    );
     }
}

:: NativeBindingX#createPlatformManager(...){
  return new PlatformManager.Builder()
                .withViewFinder(finder)
                .withDeviceResolutionTranslator(translator)
                .withViewUpdater(new PlatformManager.IViewUpdater() {
                    @Override
                    public void synchronouslyUpdateViewOnUIThread(...) {
                        NativeViewUpdateService.findUpdater(propertyName).update(
                                targetView,
                                propertyValue,
                                translator,
                                config);
                    }
                })
                .build();
}
@ NativeViewUpdateService$TranslateXUpdater implements INativeViewUpdater {
        @Override
        public void update(...) {
            if(!(cmd instanceof Double)) {
                return;
            }
            final double d = (double) cmd;
            runOnUIThread(new Runnable() {
                @Override
                public void run() {
                    targetView.setTranslationX((float) getRealSize(d,translator));
                }
            });
        }
    }
```













