## Even if you don't need Amazon to shop, be sure to collect the library!

### origin

Linkage-RecyclerView is a secondary linkage list widget developed based on the MVP architecture. It exists because of the needs of the "RxJava Magician" project.

After I found over around GitHub didn't find a suitable open source library (highly decoupled, remotely dependence), I decided to study the logic of the secondary linkage with existing open source projects and write a highly decoupled, Easy to configure, true third-party libraries that can be remotely dependent on the maven repository.

The personalized configuration of Linkage-RecyclerView is very simple. Based on MVP's “configuration decoupling” feature, users do not need to know the internal implementation details. Only by implementing the Config class can the function be customized and extended.

In addition, Linkage-RecyclerView can **run by at least one line of code** while without setting up a custom configuration.

|                           RxMagic                            |                         Eleme Linear                         |                          Eleme Grid                          |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![7.gif](https://i.loli.net/2019/05/14/5cda1fd9402ad16046.gif) | ![2.gif](https://i.loli.net/2019/05/14/5cda1fd959ed861197.gif) | ![3.gif](https://i.loli.net/2019/05/14/5cda1fd945af885525.gif) |

### aims

The goal of LinkageRecyclerView is to access the secondary linkage list by just one line of code.



In addition to one-line access and eliminating 99% of unnecessary, complex, and repetitive tasks, what extra you can get from this open source project include:

1. Neat code style and standard resource naming conventions.
2. **The best practice for writing third-party libraries by MVP architecture: users don't need to understand internal logic, and they can easily do personalized configuration by implementing interfaces.**
3. Excellent code layering and encapsulation ideas, one line of code can be accessed without any personalized configuration.
4. The main project is based on the cutting-edge JetPack MVVM architecture that follows the separation of concerns.
5. Full use of AndroidX and Material Design 2.
6. Best practices for ConstraintLayout.
7. Never use Dagger, never use geeks and write hard-coded code.



If you are thinking about **how to choose the right architecture for your project**, this project is worth your reference!



### Simple Guide:

1. Add a dependency on the library in build.gradle.

```groovy
implementation 'com.kunminx.linkage:linkage-recyclerview:1.3.5'
```

2. Prepare a bunch of JSONs according to the structure of the default grouping entity class (DefaultGroupedItem).

```java
// DefaultGroupedItem.ItemInfo Contains three fields：
String title //(Required) Title of the secondary option
String group //(Required) The name of the group in which the secondary option is located, which is the same as the title of the corresponding primary option.
String content //(optional) content of the secondary option
```

```json
[
  {
    "header": "Offer",
    "isHeader": true
  },
  {
    "isHeader": false,
    "info": {
      "content": "Good food, fattening artifact, responsive",
      "group": "Offer",
      "title": "Family bucket"
    }
  },
  {
    "header": "Selling",
    "isHeader": true
  },
  {
    "isHeader": false,
    "info": {
      "content": "Explosive models are hot",
      "group": "Selling",
      "title": "Roasted whole wings"
    }
  }
]
```

3. Introduce LinkageRecyclerView in the layout.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.kunminx.linkage.LinkageRecyclerView
        android:id="@+id/linkage"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</LinearLayout>
```

4. Parse JSON in code, and you can initialize it with at least one line of code.

```java
List<DefaultGroupedItem> items = gson.fromJson(...);

//initialize it with just one line of code
linkage.init(items);
```

Note: Since JSON entity classes are used, they must be configured to ProGuard rule before being packaged into Apk:

```java
-keep class com.kunminx.linkage.bean.** {*;}
```





### Personalized configuration:

The library prepares the Config interface (`ILevelPrimaryAdapterConfig` and `ILevelSecondaryAdapterConfig`) for the primary and secondary Adapters respectively. **When the configuration is customized, the two interfaces are implemented to replace the default configuration**.

It is set to the form of the interface, not the form of the Builder, because the linkage logic inside the secondary linkage list needs to indicate the key controls. The interface is mandatory compared to the Builder, which allows the user to understand the content that must be configured at a glance. Therefore, the interface is used to write the library through the MVP architecture.

For specific configuration, please refer to the case I wrote in `ElemeGroupedItem` and `SwitchSampleFragment`:



### Step1: Extend the entity class according to your needs

You need to extend the grouping entity class based on the requirement, based on `BaseGroupedItem`. The specific method is to write an entity class that inherits from `BaseGroupedItem`; the inner class `ItemInfo` of the entity class must also inherit from `BaseGroupedItem.ItemInfo`.

Take the Eleme grouping entity class as an example, and expand the three fields `content`, `imgUrl`, `cost`:

```jav
public class ElemeGroupedItem extends BaseGroupedItem<ElemeGroupedItem.ItemInfo> {

    public ElemeGroupedItem(boolean isHeader, String header) {
        super(isHeader, header);
    }

    public ElemeGroupedItem(ItemInfo item) {
        super(item);
    }

    public static class ItemInfo extends BaseGroupedItem.ItemInfo {
        private String content;
        private String imgUrl;
        private String cost;

        public ItemInfo(String title, String group, String content) {
            super(title, group);
            this.content = content;
        }

        public ItemInfo(String title, String group, String content, String imgUrl) {
            this(title, group, content);
            this.imgUrl = imgUrl;
        }

        public ItemInfo(String title, String group, String content, String imgUrl, String cost) {
            this(title, group, content, imgUrl);
            this.cost = cost;
        }

        public String getContent() {
            return content;
        }

        public void setContent(String content) {
            this.content = content;
        }

        public String getImgUrl() {
            return imgUrl;
        }

        public void setImgUrl(String imgUrl) {
            this.imgUrl = imgUrl;
        }

        public String getCost() {
            return cost;
        }

        public void setCost(String cost) {
            this.cost = cost;
        }
    }
}
```

Note: Since JSON entity classes are used, they must be configured to ProGuard rule before being packaged into Apk.



### Step2: Implement the interface and complete the custom configuration

When loading data and implementing a custom configuration, the generic box must indicate the entity class you wrote, noting `List<ElemeLinkageItem>`, and `new ILevelSecondaryAdapterConfig<ElemeLinkageItem.ItemInfo>()`.

```java
private void initLinkageDatas(LinkageRecyclerView linkage) {
        Gson gson = new Gson();
        List<ElemeGroupedItem> items = gson.fromJson(...);

        linkage.init(items, new ILevelPrimaryAdapterConfig() {

            private Context mContext;

            public void setContext(Context context) {
                mContext = context;
            }

            @Override
            public int getLayoutId() {
                return R.layout.default_adapter_linkage_level_primary;
            }

            @Override
            public int getTextViewId() {
                return R.id.tv_group;
            }

            @Override
            public int getRootViewId() {
                return R.id.layout_group;
            }

            @Override
            public void onBindViewHolder(
                LinkageLevelPrimaryAdapter.LevelPrimaryViewHolder holder, 
                String title, int position) {
                
                holder.getView(R.id.layout_group).setOnClickListener(v -> {
					//TODO
                });
            }

            @Override
            public void onItemSelected(boolean selected, TextView itemView) {
                itemView.setBackgroundColor(mContext.getResources().getColor(selected
                        ? com.kunminx.linkage.R.color.colorLightBlue
                        : com.kunminx.linkage.R.color.colorWhite));
                itemView.setTextColor(ContextCompat.getColor(mContext, selected
                        ? com.kunminx.linkage.R.color.colorWhite
                        : com.kunminx.linkage.R.color.colorGray));
            }

        }, new ILevelSecondaryAdapterConfig<ElemeGroupedItem.ItemInfo>() {

            private Context mContext;
            private boolean mIsGridMode;

            public void setContext(Context context) {
                mContext = context;
            }

            @Override
            public int getGridLayoutId() {
                return R.layout.adapter_eleme_secondary_grid;
            }

            @Override
            public int getLinearLayoutId() {
                return R.layout.adapter_eleme_secondary_linear;
            }

            @Override
            public int getHeaderLayoutId() {
                return R.layout.default_adapter_linkage_level_secondary_header;
            }

            @Override
            public int getTextViewId() {
                return R.id.iv_goods_name;
            }

            @Override
            public int getRootViewId() {
                return R.id.iv_goods_item;
            }

            @Override
            public int getHeaderViewId() {
                return R.id.level_2_header;
            }

            @Override
            public boolean isGridMode() {
                return mIsGridMode;
            }

            @Override
            public void setGridMode(boolean isGridMode) {
                mIsGridMode = isGridMode;
            }

            @Override
            public int getSpanCount() {
                return 2;
            }

            @Override
            public void onBindViewHolder(
                LinkageLevelSecondaryAdapter.LevelSecondaryViewHolder holder, 
                BaseGroupedItem<ElemeGroupedItem.ItemInfo> item, int position) {
                
                ((TextView) holder.getView(R.id.iv_goods_name))
                .setText(item.info.getTitle());
                
                Glide.with(mContext).load(item.info.getImgUrl())
                    .into((ImageView) holder.getView(R.id.iv_goods_img));
                
                holder.getView(R.id.iv_goods_item).setOnClickListener(v -> {
                    //TODO
                });

                holder.getView(R.id.iv_goods_add).setOnClickListener(v -> {
                    //TODO
                });
            }
        });
```



# Thanks to

[material-components-android](https://github.com/material-components/material-components-android)

[AndroidX](https://developer.android.google.cn/jetpack/androidx)



# My Pages

Email：[kunminx@gmail.com](mailto:kunminx@gmail.com)

Blog：[KunMinX 的个人博客](https://kunminx.github.io/)

Juejin：[KunMinX 在掘金](https://juejin.im/user/58ab0de9ac502e006975d757/posts)

<span id="wechatQrcode">KunMinX's WeChat Public Account（微信公众号）：</span>

![公众号](https://upload-images.jianshu.io/upload_images/57036-dc3af94a5daf478c.jpg)

# License

```
Copyright 2018-2019 KunMinX

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
