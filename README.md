
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

%matplotlib inline
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

path = './data/UserBehavior.csv'
data_user = pd.read_csv(path)
cols = ['UserID', 'ItemID', 'CatogoryID', 'BehaviorType', 'TimeStamps']
data_user.columns = cols
data_user.head()
```

![UserID	ItemID	CatogoryID	BehaviorType	TimeStamps
0	1	2333346	2520771	pv	1.511562e+09
1	1	2576651	149192	pv	1.511573e+09
2	1	3830808	4181361	pv	1.511593e+09
3	1	4365585	2520377	pv	1.511596e+09
4	1	4606018	2735466	pv	1.511616e+09](https://img-blog.csdnimg.cn/20200428182653709.PNG)




```python
data_user.apply(lambda x: sum(x.isnull()))
```
![UserID          0
ItemID          0
CatogoryID      0
BehaviorType    0
TimeStamps      1
dtype: int64](https://img-blog.csdnimg.cn/20200428182746692.PNG)



```python
import time

def get_unixtime(timeStr):
    formatStr = "%Y-%m-%d %H:%M:%S"
    tmObject = time.strptime(timeStr, formatStr)
    tmStamp = time.mktime(tmObject)
        
    return int(tmStamp)
    
# 数据集描述的时间范围
startTime = get_unixtime("2017-11-25 00:00:00")
endTime = get_unixtime("2017-12-3 23:59:59")

# 筛选出符合时间范围的数据
data_user['TimeStamps'] = data_user['TimeStamps'].astype('int64')
data_user = data_user.loc[(data_user['TimeStamps'] >= startTime) & (data_user['TimeStamps'] <= endTime)]
```


```python
#时间处理
data_user['time'] = data_user['TimeStamps'].apply(lambda t: time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(t)))
data_user['date'] = data_user['time'].str[0:10]
data_user['hour'] = data_user['time'].str[11:13].astype(int)
data_user['date'] = pd.to_datetime(data_user['date'])

data_user.head()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428182859412.PNG)


```python
data_user_buy1 = data_user[data_user.BehaviorType == 'buy'].groupby(['date','UserID']).count()['BehaviorType'].reset_index().rename(columns={'BehaviorType':'total'})

data_user_buy2 = data_user_buy1.groupby('date').sum()['total'] / data_user_buy1.groupby('date').count()['total']

plt.figure(figsize=(10,7))
data_user_buy2.plot()
plt.ylabel('日ARPPU')
plt.title('ARPPU变化情况')
plt.savefig('ARPPU变化情况')

```python
data_user['operation'] = 1
data_user_buy2 = data_user.groupby(['date', 'UserID', 'BehaviorType'])['operation'].count().reset_index().rename(columns = {'operation':'total'})

#每天消费总次数/每天总活跃人数
data_user_buy2.groupby('date').apply(lambda x: x[x['BehaviorType'] == 'buy'].total.sum()/len(x.UserID.unique()) ).plot()
plt.ylabel('日ARPU')
plt.title('ARPU变化情况')
plt.savefig('ARPU变化情况')
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429160428770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

```python

data_user_buy2.groupby('date').apply(lambda x: x[x['BehaviorType'] == 'buy'].total.count()/len(x.UserID.unique()) ).plot()
plt.ylabel('付费率')
plt.title('付费率变化情况')
plt.savefig('付费率变化情况')
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429160413221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)



```python
# 多子图绘制 如：将上面用到的图形一起绘制
# 导入subplots（类似matplotlib）
from plotly.subplots import make_subplots

labels = df_userbehavior['behavior']

# Create subplots: use 'domain' type for Pie subplot
fig = make_subplots(rows=1, cols=2, specs=[[{'type':'domain'}, {'type':'domain'}]])
fig.add_trace(go.Pie(labels=labels, values=data_user_count.values, name="淘宝用户行为"),
              1, 1)
fig.add_trace(go.Pie(labels=labels, values=df_userbehavior['count'], name="淘宝独立用户行为"),
              1, 2)

# Use `hole` to create a donut-like pie chart
fig.update_traces(hole=.4, hoverinfo="label+percent+name")

fig.update_layout(
    title_text="淘宝用户行为情况 | 左：淘宝用户行为， 右：淘宝独立用户行为",
    # Add annotations in the center of the donut pies.
    annotations=[dict(text='非独立', x=0.18, y=0.5, font_size=20, showarrow=False),
                 dict(text='独立', x=0.8, y=0.5, font_size=20, showarrow=False)])
fig.show()
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200430152910340.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)


```python
from pyecharts import options as opts
from pyecharts.charts import Funnel
from pyecharts.faker import Faker

attr = ['浏览', '放入购物车', '收藏', '购买']
value = [3431904, 213634, 111140, 76707]    #这里有个bug
funnel = Funnel()
funnel.add("淘宝用户行为", [list(z) for z in zip(attr, value)])
funnel.set_global_opts(title_opts=opts.TitleOpts(title="淘宝用户行为"))
funnel.render("funnel_base.html") 
funnel.render_notebook()
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429202820614.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)


```python
from datetime import timedelta

#建立n日留存率计算函数
def cal_retention(data,n): #n为n日留存
    user=[]
    date=pd.Series(data.date.unique()).sort_values()[:-n] #时间截取至最后一天的前n天
    retention_rates=[]
    new_users=[]
    retention_user=[]
    for i in date:
        new_user=set(data[data.date==i].UserID.unique())-set(user) #识别新用户，本案例中设初始用户量为零
        user.extend(new_user)  #将新用户加入用户群中
        #第n天留存情况
        user_nday=data[data.date==i+timedelta(n)].UserID.unique() #第n天登录的用户情况
        a=0
        for UserID in user_nday:
            if UserID in new_user:
                a+=1
        b = len(new_user)
        retention_rate=a/b #计算该天第n日留存率
        retention_rates.append(retention_rate) #汇总n日留存数据
        new_users.append(b) #汇总n日的新用户数
        retention_user.append(a) #汇总n日留存的用户数
    data_new_user = pd.Series(new_users, index=date)
    data_retention_user = pd.Series(retention_user, index=date)
    data_retention_rate = pd.Series(retention_rates,index=date)
    data_retention = pd.concat([data_new_user,data_retention_user,data_retention_rate], axis=1)
    data_retention.columns=['new_user','retention_user','retention_rate']
    return data_retention

data_retention1=cal_retention(data_user,1)
data_retention2=cal_retention(data_user,2)
data_retention6=cal_retention(data_user,6)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429202657966.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429160546213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429144949479.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

```python
data_rebuy[data_rebuy>=2].count()/data_rebuy.count()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429221206329.PNG)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429221828232.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)



```python
pv_daily = data_user.groupby('date').count()['UserID']
uv_daily = data_user.groupby('date')['UserID'].apply(lambda x: x.drop_duplicates().count())

pv_uv_daily = pd.concat([pv_daily,uv_daily], axis=1)
pv_uv_daily.columns=['pv','uv']
pv_uv_daily
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428182946544.PNG)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428183309451.PNG)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042818371289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428183945718.PNG)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428183945615.PNG)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428184115560.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200428185100247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429205729118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

```python
trace_basic = [go.Bar(x = rfm['rank'].value_counts().index,
                     y = rfm['rank'].value_counts().values,
                     marker = dict(color='orange'), opacity=0.50)]
layout = go.Layout(title='用户等级情况', xaxis=dict(title='用户重要度'))
figure_basic = go.Figure(data=trace_basic, layout=layout)
figure_basic

trace = [go.Pie(labels=rfm['rank'].value_counts().index,
                values = rfm['rank'].value_counts().values,
               textfont = dict(size=12,color='white'))]
layout = go.Layout(title='用户等级比例')
figure_pie = go.Figure(data=trace, layout=layout)
figure_pie
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429134429735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429134429728.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

```python
fig = plt.figure(figsize=(16,12))
#柱形图
ax1 = fig.add_subplot(111)
ax1.bar(data_item_count.index, data_item_count.values)
for a,b in zip(data_item_count.index,data_item_count.values):
    plt.text(a, b+100,'%s'% b, ha='center', va= 'bottom',fontsize=10)


#平滑化
from scipy import interpolate

x = data_item_count.index
y = df_item_count['percentage']
tck = interpolate.splrep(x, y, s=0)
xnew = np.linspace(x.min(),x.max(),100)
ynew = interpolate.splev(xnew, tck, der=0) 

#折线图
ax2 = ax1.twinx()
ax2.plot(xnew, ynew, label="percentage", color='red')

ax1.set_ylabel('商品数目')
ax2.set_ylabel('所占百分比')
ax2.set_xlabel('购买次数')
plt.title('商品销售分布', fontsize=25)
plt.savefig('商品销售分布')
plt.show()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429134737600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

```python
ax1 = df_catogory_buy[['buy', 'fav', 'cart', 'pv']].plot.bar()

ax2 = ax1.twinx()
df_catogory_buy.index = df_catogory_buy.index.astype(str)
ax2.plot(df_catogory_buy.index, df_catogory_buy[['buy/pv']])

ax1.set_ylabel('次数')
ax2.set_ylabel('转化率')
plt.title('购买次数前二十的商品种类')
plt.savefig('购买次数前二十的商品种类')
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429134857925.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

```python
import plotly.express as px
fig = px.treemap(
    df_buy, path=['CatogoryID'], values='购买次数', title='购买次数前二十的商品种类'
)
fig.show() 

fig = px.treemap(
    df_item_buy, path=['CatogoryID','ItemID'], values='count', title='商品购买情况(销量前100)'
)
fig.show() 
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429150740642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200429154952128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTM5OTA3NA==,size_16,color_FFFFFF,t_70)

