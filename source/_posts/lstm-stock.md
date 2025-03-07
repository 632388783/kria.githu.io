---
title: 学习 LSTM 算法做的的股票价格预测demo
index_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
banner_img: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
cover: https://qingshaner.oss-cn-hangzhou.aliyuncs.com/images/202204031102225.jpg
date: 2025-03-05 18:43:31
tags: [LSTM, Python]
categories: [机器学习, Python]
toc: true
permalink: /lstm-stock-prediction/
description: 使用 LSTM（长短期记忆）算法来预测股票价格的demo
comment: 'valine'
---

# 基于 LSTM 算法的股票价格预测学习教程

今天终于迎来了探索用 LSTM（长短期记忆）算法预测股票价格的学习之旅，内心满是期待与激动！一直对股票市场的复杂波动充满好奇，而现在有机会通过前沿的算法来揭开它神秘的面纱，想想就觉得超酷。我们即将借助 Tushare 获取宝贵的股票数据，再依靠 Keras 搭建起神奇的 LSTM 模型，一步步精心训练它，让它逐渐拥有预测股票价格的强大能力，感觉每一步都像是在开启一场充满惊喜的冒险。

## 环境配置

在踏上这段奇妙学习征程之前，必须得精心准备好我们的 “学习装备”，也就是要确保成功安装好以下这些 Python 库。它们可都是我们在这场学习冒险中不可或缺的得力助手，每一个都有着独特的作用。

`tushare`：这简直就是获取股票数据的 “超级神器”！有了它，我们就能便捷地从海量的金融数据中，精准捞出我们所需的股票信息，为后续的分析和模型训练奠定坚实基础。

`numpy`：处理数值计算方面，它堪称一把无可比拟的 “利刃”。在数据处理的各个环节，从简单的数组运算到复杂的矩阵操作，都离不开它的帮助。

`scikit - learn`：在机器学习领域，它可是大名鼎鼎的常用库。里面提供了丰富多样的工具和算法，无论是数据预处理中的特征提取、数据降维，还是模型训练中的分类、回归算法，都能在其中找到对应的实现。

`keras`：搭建神经网络模型的过程中，它绝对是我们的 “最佳拍档”。它以简洁高效的方式，让搭建复杂的神经网络模型变得不再那么遥不可及。使用它，我们可以轻松地构建各种层，组合出不同结构的模型。

`matplotlib`：这个库的作用太关键了！它能够将我们处理和分析的数据，以直观、形象的图形展示出来。通过各种图表，我们可以一目了然地看到数据的分布、趋势以及模型的预测结果等。
`pandas`：对于处理表格形式的数据，`pandas`可以轻松地对这些数据进行清洗、筛选、分析和处理。

安装这些库其实并没有想象中那么困难，就像在魔法世界里念出一句简单的咒语。在命令行中输入下面这行神奇的命令，然后耐心等待魔法生效（也就是等待安装完成）就好啦。

```bash
pip install tushare numpy scikit - learn keras matplotlib pandas
```

安装过程中，这些库之间可能存在版本兼容性的问题。

## 获取股票数据

要让我们精心构建的模型学会预测股票价格，首先得为它提供充足且优质的学习 “素材”，这就轮到 Tushare API 闪亮登场啦。不过，在使用这个强大工具之前，我们需要在 Tushare 平台上注册一个属于自己的账号，并成功获取到 API 密钥。这就好比我们要进入一座神秘的宝藏库，而这个密钥就是那把开启宝藏大门的关键钥匙。

```python
import tushare as ts
import pandas as pd

# 设置Tushare API密钥，这里一定要填入自己辛苦申请到的密钥哦
ts.set_token('your_tushare_api_key')

# 初始化Tushare接口，准备好开启数据获取的奇妙之旅啦
pro = ts.pro_api()

def fetch_stock_data(stock_code, start_date, end_date):
    df = pro.daily(ts_code=stock_code, start_date=start_date, end_date=end_date)
    df = df.sort_values('trade_date')
    df.set_index('trade_date', inplace=True)
    df.index = pd.to_datetime(df.index)
    return df

stock_data = fetch_stock_data('002439.SZ', '20000101', '20250304')
print(stock_data.head())
```

当运行这段代码的那一刻，感觉就像是在向神秘的数据宝库发出请求，然后看着指定股票在特定时间段内的数据源源不断地呈现在眼前，真的太神奇了！不过，我也在思考，要是获取的数据量非常大，会不会对内存和处理速度造成压力呢？该如何优化数据获取和存储的过程呢？这些问题都有待在后续学习中进一步探索。

## 数据预处理

虽然我们已经成功获取到了宝贵的数据，但可不能直接就把它们丢给模型去学习哦，就如同我们吃水果之前，必须先把水果洗干净、处理好一样，数据也需要进行一系列的预处理。我们首先要对股票收盘价进行归一化处理，这一步操作就像是把各种不同大小、形状的水果，都统一放进一个标准尺寸的盒子里，让它们变得整齐、规整，这样模型在处理数据时会更加轻松、高效。然后，我们还要创建一个时间序列数据集，这个数据集对于模型来说，就像是一份精心准备的学习资料，里面蕴含着数据随时间变化的规律，是模型学习预测的重要 “营养来源”。

```python
import numpy as np
from sklearn.preprocessing import MinMaxScaler

def preprocess_stock_data(df):
    data = df[['close']].values
    scaler = MinMaxScaler(feature_range=(0, 1))
    scaled_data = scaler.fit_transform(data)
    timestamps = df.index
    return scaled_data, scaler, timestamps

scaled_data, scaler, timestamps = preprocess_stock_data(stock_data)
```

完成这一步后，看着处理后的数据，感觉它们已经从原始的 “粗糙模样”，变得更加 “精致”，更适合模型去 “消化吸收” 了。但我也在思考，除了这种归一化方法，还有没有其他更好的方式来处理数据，以提高模型的性能呢？时间序列数据的创建过程中，不同的时间步长选择又会对模型产生怎样的影响呢？这些问题都值得深入研究。

## 创建时间序列数据集

现在，我们要着手创建一个时间序列数据集，这个数据集对于模型而言，就像是一张详细的学习路线图，它清晰地告诉模型如何根据过去的数据信息，去预测未来的趋势。

```python
def create_dataset(data, time_step):
    X, y = [], []
    for i in range(len(data) - time_step):
        X.append(data[i:(i + time_step)])
        y.append(data[i + time_step])
    return np.array(X), np.array(y)
```

通过编写这段代码，我们就像是在为模型精心准备一份独特的学习资料，将数据按照时间顺序进行合理的组织和划分。但在编写和理解这段代码的过程中，我也在思考，这个时间步长的选择有没有什么科学的方法或者经验法则呢？不同的时间步长设置，会对模型的预测精度和泛化能力产生怎样的影响呢？看来这又是一个需要深入探索的方向。

## 构建和训练 LSTM 模型

终于到了整个学习过程中最激动人心的环节 —— 搭建模型！我们将使用 Keras 来构建一个强大的 LSTM 模型。这个模型就像是我们精心培养的一个 “超级大脑”，我们要通过一系列的训练，让它逐渐学会预测股票价格的神奇本领。

```python
from keras.models import Sequential
from keras.layers import LSTM, Dense

def build_lstm_model(input_shape):
    model = Sequential()
    model.add(LSTM(100, return_sequences=True, input_shape=input_shape))
    model.add(LSTM(100))
    model.add(Dense(1))
    model.compile(optimizer='adam', loss='mse')
    return model

input_shape = (X.shape[1], X.shape[2])
model = build_lstm_model(input_shape)

# 划分训练集和测试集，这一步就像是把我们手中的练习题，合理地分成两部分，一部分用来学习解题方法，另一部分用来检验我们的学习成果
train_size = int(len(X) * 0.8)
X_train, X_test = X[:train_size], X[train_size:]
y_train, y_test = y[:train_size], y[train_size:]

model.fit(X_train, y_train, epochs=10, batch_size=32, verbose=1)
model.save('lstm_stock_model.h5')
```

在构建模型的过程中，每添加一层，每设置一个参数，都像是在为这个 “超级大脑” 塑造独特的思维方式。看着模型在训练过程中，参数不断地调整和优化，就好像看到自己的 “小徒弟” 在努力学习、不断进步，内心充满了期待。但同时，我也在思考，模型中的这些超参数，比如 LSTM 层的神经元数量、训练的轮数、批次大小等，它们的取值对模型性能的影响究竟有多大呢？有没有一种更科学的方法来选择这些超参数，以达到最优的模型性能呢？这无疑是后续学习中需要重点攻克的难题。

## 预测和评估模型

经过一番精心的训练，我们的模型终于 “学有所成”，现在是时候检验它的学习成果啦。我们将使用训练好的模型来进行预测，然后通过科学的评估方法，来判断模型的性能究竟如何，看看它的预测结果与实际情况到底有多接近。

```python
from sklearn.metrics import mean_squared_error

predictions = model.predict(X_test)
predictions = scaler.inverse_transform(predictions)
y_test = scaler.inverse_transform(y_test)

mse = mean_squared_error(y_test, predictions)
print(f'Mean Squared Error: {mse}')
```

通过计算均方误差，我们得到了一个量化的指标，它直观地反映了模型预测结果与实际结果之间的差距。看到这个结果的那一刻，我就在想，这个误差值在实际应用中意味着什么呢？如果误差较大，我们该从哪些方面去改进模型呢？是调整模型结构，还是优化数据处理过程，亦或是改变训练参数呢？这些问题都需要我们深入思考和分析。

## 可视化结果

单纯地看数字可能不够直观，无法让我们全面、深入地了解模型的表现。这时候，就该轮到`matplotlib`库大显身手啦！我们将借助它的强大功能，把实际股价、预测股价以及未来预测股价，以直观、形象的图表形式展示出来，这样我们就能一目了然地看到模型的预测效果，发现其中的规律和问题。

```python
import matplotlib.pyplot as plt

def plot_results(actual_prices, predicted_prices, future_predictions=None, timestamps=None):
    plt.figure(figsize=(14, 5))
    if timestamps is not None:
        plt.plot(timestamps[:len(actual_prices)], actual_prices, color='blue', label='Actual Stock Price')
        test_timestamps = timestamps[-len(predicted_prices):]
        plt.plot(test_timestamps, predicted_prices, color='red', label='Predicted Stock Price')
        if future_predictions is not None:
            future_steps = len(future_predictions)
            future_dates = pd.date_range(start=test_timestamps[-1], periods=future_steps + 1, freq='B')[1:]
            plt.plot(future_dates, future_predictions, color='green', label='Future Predictions')
    else:
        plt.plot(actual_prices, color='blue', label='Actual Stock Price')
        plt.plot(predicted_prices, color='red', label='Predicted Stock Price')
        if future_predictions is not None:
            future_steps = len(future_predictions)
            plt.plot(range(len(actual_prices), len(actual_prices) + future_steps),
                     future_predictions, color='green', label='Future Predictions')

    plt.title('Stock Price Prediction and Future Forecast')
    plt.xlabel('Time')
    plt.ylabel('Stock Price')
    plt.legend()
    plt.grid(True)
    plt.show()

plot_results(y_test, predictions, timestamps=timestamps[train_size + seq_length:])
```

当看到这些图表出现在眼前的那一刻，感觉就像是打开了一扇通往模型内心世界的窗户，我们可以清晰地看到模型的预测轨迹与实际股价的对比情况。通过观察图表，我们可以发现模型在哪些时间段预测得比较准确，哪些地方还存在较大的偏差。

## 预测未来趋势

最后，我们将使用训练好的模型，来大胆预测未来几个月的股价走势。这就像是让我们精心培养的 “超级大脑”，对未来的股票市场进行一次 “占卜”，看看它能否准确地洞察未来的趋势。

```python
def predict_future_trend(model, data, scaler, seq_length, future_steps):
    future_preds = []
    current_sequence = data[-seq_length:]

    for _ in range(future_steps):
        prediction = model.predict(current_sequence[np.newaxis, :, :])
        future_preds.append(prediction[0, 0])
        current_sequence = np.append(current_sequence[1:], prediction, axis=0)

    future_preds = np.array(future_preds).reshape(-1, 1)
    future_predictions = scaler.inverse_transform(future_preds)
    return future_predictions

future_steps = 3 * 20
future_predictions = predict_future_trend(model, scaled_data, scaler, seq_length, future_steps)

plot_results(y_test, predictions, future_predictions=future_predictions, timestamps=timestamps[train_size + seq_length:])
```

在等待模型预测结果的过程中，内心充满了紧张和期待。当看到预测结果呈现出来的那一刻，我在想，这些预测结果的可信度究竟有多高呢？如果将这些预测结果应用到实际的股票投资决策中，还需要考虑哪些因素呢？

PS: 模型预测结果可能存在一些偏差，需要根据实际情况进行校正。该文档仅作为学习参考。
