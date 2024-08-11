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

