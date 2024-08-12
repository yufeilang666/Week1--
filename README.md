# Week1--
Week1-按照要求进行爬虫和量化选股


-从“我的研究”开始，导出股票到excel文档：前三个交易日涨跌幅在-5到15之间，10日均线斜率大于-5的股票，并且列出前三个交易日区间涨幅，后一个交易日最大涨幅，涨跌幅，并按成交额排序。 
用问财选股函数来获取上述股票，“成交额排序“，”前3个交易日区间涨幅在-10%和15%之间“，”10日均线斜率大于-5“，”前三个交易日区间涨幅“，”后一个交易日最大涨幅“，”涨跌幅“

df = query_iwencai("成交额排序, 前3个交易日区间涨幅在-10%和15%之间, 10日均线斜率大于-5, 前三个交易日区间涨幅, 后一个交易日最大涨幅, 涨跌幅")
df = pd.DataFrame(df)
df.to_excel('output.xlsx', index=False, engine='xlsxwriter')
导出股票链接🔗如下
[output.xlsx](https://github.com/user-attachments/files/16570355/output.xlsx)



-导出2023年全部的地天板股票，展示其次日涨跌幅，后3日涨跌幅  代码如下

# 获取所有股票的基本信息
all_stocks = get_all_securities(ty='stock')
#all_stocks
# 提取股票代码
stock_codes = all_stocks.index.tolist()
stock_codes


# 获取2023年的所有股票信息
data = get_price(stock_codes, '20230101 09:00', '20231231 16:00', '1d', ['close', 'open', 'low', 'high'])


dtb_stocks = {} # 创建空的结果字典，用于保存每只股票的次日和后三日涨跌幅
    
for stock, df in data.items():
    # 筛选地天板条件：开盘价等于最低价，收盘价等于最高价
    is_dtb = (df['open'] == df['low']) & (df['close'] == df['high'])
    
    # 获取符合条件的日期和对应数据
    dtb_dates = df[is_dtb]
    
    if not dtb_dates.empty:
        dtb_stocks[stock] = dtb_dates       
    
    
######
# 创建一个新的字典results用于存储符合要求的股票的次日涨跌幅和后三日涨跌幅
results = {}

# 计算次日涨跌幅和后三日涨跌幅   
for stock, df in dtb_stocks.items():
    # 计算次日涨跌幅
    df['next_day_return'] = (df['close'].shift(-1) - df['close']) / df['close'] * 100
    
    # 计算后三日涨跌幅
    df['three_days_return'] = (df['close'].shift(-3) - df['close']) / df['close'] * 100
    
    # 保存结果
    results[stock] = df[['next_day_return', 'three_days_return']]   
    
####   

# 将所有结果合并为一个DataFrame，便于后续分析或导出
final_df = pd.concat(results)

# 展示结果
final_df.head()

# 如果需要将结果保存为Excel文件
final_df.to_excel("all_stocks_returns_2023.xlsx")
[all_stocks_returns_2023.xlsx](https://github.com/user-attachments/files/16575650/all_stocks_returns_2023.xlsx)


### -导出在2023年，选股指标为“10：00前成交额排序，当日涨跌幅，5日均线斜率>-10，前三个交易日区间涨幅，后一个交易日最大涨幅，后一个交易日涨幅，前三个交易日区间涨幅在-10%到15%之间”，取每个交易日前两个数据，输出到excel!
df2 = query_iwencai("10：00前成交额排序, 涨跌幅, 5日均线斜率>-10, 前三个交易日区间涨幅, 后一个交易日最大涨幅, 后一个交易日涨幅, 前三个交易日区间涨幅在-10%到15%之间")
df2 = pd.DataFrame(df2) 
df2.to_excel('output2.xlsx', index=False, engine='xlsxwriter')
[output2.xlsx](https://github.com/user-attachments/files/16575651/output2.xlsx)


######
任务3：导出2023年5连板及以上的股票数据，包括“股票名称	连板次数	断板时间	断板前后最高价	断板回调到最低价的时间	断板回调的最低价	回调百分比	第一次反弹的最高价	反弹的比率	第一次反弹的时间”


# 获取所有股票代码及名称
all_stocks = get_all_securities(ty='stock')

# 存储结果的列表
results = []

# 遍历所有股票
for stock_code in all_stocks.index:
    stock_name = all_stocks.loc[stock_code]['display_name']
    
    # 获取2023年的历史价格数据
    data = get_price(stock_code, start_date='20230101', end_date='20231231', 
                     fields=['open', 'close', 'high', 'low'], fre_step='1d')
    
    # 识别连板次数（假设涨停是指某个百分比的涨幅，例如 10%）
    data['limit_up'] = data['close'].pct_change().ge(0.1)  # 这里使用10%作为涨停的例子
    
    # 计算连板的长度
    data['consecutive_limits'] = data['limit_up'].cumsum() - data['limit_up'].cumsum().where(~data['limit_up']).ffill().fillna(0)
    
    # 筛选5连板及以上的情况
    five_limit_days = data[data['consecutive_limits'] >= 5]
    
    if not five_limit_days.empty:
        # 找到断板的时间和价格变化
        for i in range(len(five_limit_days)):
            # 找到断板时间
            try:
                limit_end = five_limit_days.index[i + 1]
            except IndexError:
                continue
            
            # 获取断板前后的最高价、最低价等信息
            pre_high = data.loc[:limit_end, 'high'].max()
            post_low = data.loc[limit_end:, 'low'].min()
            post_low_time = data.loc[limit_end:, 'low'].idxmin()
            
            # 计算回调百分比
            pullback_percentage = (pre_high - post_low) / pre_high * 100
            
            # 找到第一次反弹的最高价及反弹率
            rebound_high = data.loc[post_low_time:, 'high'].max()
            rebound_percentage = (rebound_high - post_low) / post_low * 100
            rebound_time = data.loc[post_low_time:, 'high'].idxmax()
            
            # 存储结果
            results.append({
                '股票名称': stock_name,
                '连板次数': data['consecutive_limits'].max(),
                '断板时间': limit_end,
                '断板前后最高价': pre_high,
                '断板回调到最低价的时间': post_low_time,
                '断板回调的最低价': post_low,
                '回调百分比': pullback_percentage,
                '第一次反弹的最高价': rebound_high,
                '反弹的比率': rebound_percentage,
                '第一次反弹的时间': rebound_time
            })

# 转换为DataFrame并导出结果
results_df = pd.DataFrame(results)
results_df.to_excel("five_limit_stocks_2023.xlsx", index=False)
# csv导出文件如下
[five_limit_stocks_2023.xlsx](https://github.com/user-attachments/files/16580910/five_limit_stocks_2023.xlsx)


# 打印结果
print(results_df)









