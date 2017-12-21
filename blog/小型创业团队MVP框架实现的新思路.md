---
title: 小型创业团队MVP框架实现的新思路
date: 2017-07-15 22:20:08
tags: [Android,架构]
categories: Android
---

# 前言

该框架设计源于MVP，但做了一定程度上的精简，窃以为非常适合小步快跑的小型创业团队使用。核心思想仍旧是将视图层和数据层分离，增强了项目的可扩展性，也使项目的整体结构更加清晰明了。

# 架构

## View层

* 与原MVP框架中的View层基本保持一致
* 只处理view层的相关操作，原则上不处理纯数据逻辑层的任何操作
* 每一个View层都需要抽象出一个View层接口，对应的Activity和Fragment作为其实现类，如`BaseActivity implements IBaseView`

## Presenter层

* 只处理数据操作，处理业务逻辑，原则上不处理任何view层的相关操作
* 如果需要更新视图（如获取网络数据并显示内容），则只需要传递必要参数，并通知view层更新即可
* 通常情况下，Presenter的构造参数有两个：Context和View层接口，例如，`BasePresenter presenter = new BasePresenter(context, iBaseView)`。这里需要额外说明的是，多数的MVP框架中，Presenter层是没有直接传入Context变量的，本框架中，由于Model层的特殊性，Context变量主要用于Model层使用

## Model层

* 与传统意义上的MVP框架中的Model层相比较，本框架中的Model层有很大的不同。
* 本框架中，Model层不再是由一个单独的类来实现，而是根据细化的功能点，进行逐个封装，变成一个又一个管理类（Manager），类似于“群岛”的概念。例如，专一处理网络层的`VolleyManager`，专一处理GSON解析的`GsonManager`等等
* Presenter层中直接调用Model层各Manager的静态方法，或者是实现Manager的接口来完成相关的数据操作
* 这样处理的优点是为了极大的减少开发工作量，且仍旧要保持框架的核心思想（视图层和数据层分离）不动摇，在本框架中，实现一个具体的需求点，只需要新建Activity类、View接口、Presenter类这3个核心文件即可完成该功能点的项目代码搭建，不再是按部就班的建立诸多额外的文件，导致项目过于庞大臃肿

# Demo

## View层

### 接口类

```
public interface IReportView {

    void showScore(String score);

    void showStatistics(String defeat, String avg, String best);

}
```

### Activity（实现类）

```
public class ReportActivity implements IReportView {

    private ReportPresenter mPresenter;
    private TextView mTvScore;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_report);
        initView();
        initData();
    }

    private void initData() {
        mPresenter = new ReportPresenter(this, this);
        mPresenter.mPaperId = getIntent().getIntExtra(INTENT_PAPER_ID, 0);

        mPresenter.getData();
    }

    @Override
    public void showScore(String score) {
        mTvScore.setText(score);
    }

    @Override
    public void showStatistics(String defeat, String avg, String best) {
        ViewStub vs = (ViewStub) findViewById(R.id.report_statistics_vs);
        if (vs == null) return;
        vs.inflate();

        TextView tvDefeat = (TextView) findViewById(R.id.report_statistics_defeat);
        TextView tvAvg = (TextView) findViewById(R.id.report_statistics_avg);
        TextView tvBest = (TextView) findViewById(R.id.report_statistics_best);

        if (tvDefeat != null) {
            defeat = defeat + "%";
            tvDefeat.setText(defeat);
        }

        if (tvAvg != null) {
            avg = avg + "分";
            tvAvg.setText(avg);
        }

        if (tvBest != null) {
            best = best + "分";
            tvBest.setText(best);
        }
    }

}

```

## Presenter层

```
public class ReportPresenter implements RequestCallback {

    private IReportView mIView;

    public int mPaperId;

    public ReportPresenter(Context context, IMeasureMockReportView view) {
        super(context);
        mIView = view;
        mRequest = new Request(context, this);
    }

    public void getData() {
        mRequest.getDetail(mPaperId);
    }

    @Override
    public void requestCompleted(JSONObject response, String apiName) {
        if (HISTORY_EXERCISE_DETAIL.equals(apiName)) {
            dealResponse(response);
        }
    }

    private void dealResponse(JSONObject response) {
        ReportResp resp = GsonManager.getModel(response, ReportResp.class);
        if (resp == null) return;
        
        // 显示分数
        mIView.showScore(String.valueOf(resp.getScore()));

        // 显示统计信息
        showStatistics(resp.getRank());
    }

    private void showStatistics(ReportResp.RankBean rankBean) {
        if (rankBean == null || !rankBean.isAvailable()) return;

        double defeat = mockRankBean.getDefeat() * 100;
        BigDecimal bigDecimal = new BigDecimal(defeat);
        defeat = bigDecimal.setScale(1, BigDecimal.ROUND_HALF_UP).doubleValue();

        mView.showStatistics(
                String.valueOf(defeat),
                String.valueOf(rankBean.getAvg()),
                String.valueOf(rankBean.getTop()));
    }

}
```

## Model层

* `RequestCallback`为`VolleyManger`的接口
* `GsonManager`为解析GSON的管理类

# 扩展

## 大型模块的实现思路

* 项目中总会存在各种各样的大型模块，如果将全部的业务逻辑都简单的放在上述3个文件中，显然不现实，会造成项目的臃肿化
* 此时，需要将大型模块进行分解，拆分成一个个的子模块，子模块本身是一套相对独立的MVP架构
* 为了保证父模块与子模块间的交流和管理，子模块的MVP三层需要全部继承自父模块的MVP三层，例如，DemoAPresenter extends BasePresenter

## 项目迭代扩展的实现思路

* 很多时候，会有针对某成熟模块进行业务需求的扩展，如果每次都在原有模块上修改（例如，简单的增加if-else判定），会无形中增加开发成本和测试成本
* 此时，需要将扩展前后的需求点进行抽离，新建统一的Base类来管理扩展前后中相同的功能点，扩展的同时无需对原有模块做额外的测试
* 例如，项目中原有一个普通的报告页面（对应P层`ReportPresenter`），此时，如果想扩展多个新的报告页面，且都包含显示统计信息的方法（`showStats()`），那么正确的做法应该是将P层抽离出基类（`ReportBasePresenter`），将`showStats()`方法移至基类中实现，另外`ReportPresenter`和扩展的页面（如`ReportAPresenter`、`ReportBPresenter`）都统一继承自`ReportBasePresenter`

# 后记

框架本身没有绝对的好与坏，只存在是否适合当前的需求。框架本身也不是一成不变的，随着不断的迭代，必须随时总结随时完善。只有这样，才能使框架本身所带来的价值最大化。

# 参考资料

[Google官方的MVP框架开源代码](https://github.com/googlesamples/android-architecture)
