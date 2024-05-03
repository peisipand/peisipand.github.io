---
layout:     post
title:      使用 MATLAB 绘制拟合线周围的 95% 置信区间
subtitle:   Plotting 95% Confidence Intervals Around Fitted Line Using MATLAB
date:       2024-04-29
author:     Peisipand
header-img: img/wallhaven-dg3opm.jpg
catalog: true
tags:
    - Matlab
    - Plot
---


---

读论文的时候发现一个好看的图（如下），之前知道这种图用R语言来画非常方便，这次想用MATLAB来实现一下。

![picture1](/img/Matlab_Plotting_95CI/reference.png "图1")

```
以下是我写的自动从网上获取ncl colormap RGB值的函数，该函数也可以从[我的github](https://github.com/peisipand/matlab_function_by_pzp/blob/main/ncl_colormap.m)中获取。

```bash
clc;clear;
x = [10,20,50,300 400 500]'; % in_situ_emission_rate
y = [15,24,60,304 402 400]'; % estimated_emission_rate
e = std(y)*ones(size(y))/10; % e是误差值，这里我们使用标准差作为示例
% 给x进行排序，方便绘制区间图
[X, I] = sort(x);
Y = y(I);
x = X;
y = Y;
mdl = fitlm(x,y);%求一元线性拟合的参数
[p, s] = polyfit(x, y, 1);
y1 = polyval(p, x);
[yfit, dy] = polyconf(p, x, s, 'predopt', 'curve');
r2 = ['R^2=',num2str(mdl.Rsquared.Ordinary,'%.2f')];% 即一元线性拟合的R平方
fitted_line = ['y=',num2str(p(1),'%.2f'),'x+',num2str(p(2),'%.2f')];
% 开始绘图
figureHandle = figure;
set(gcf,'unit','centimeters','position',[25,15,8.5,8.5]); % 图形窗口在电脑屏幕上的位置和尺寸[左 下 宽 高] 
% 单栏图宽度为 8.5 cm,双栏为17.4 mm
p1 = patch([x; flipud(x)], [yfit - dy; flipud(yfit + dy)], [217 235 249] / 255, 'FaceA', 0.95, 'EdgeA', 0);
hold on;
p2 = plot(x, y1, 'Color', [20 111 170] / 255, 'linewidth', 1);
hold on;
% p3 = scatter(x, y, 'filled','MarkerFaceColor',[20 111 170] / 255);
p3 = errorbar(x, y, e, 'o','MarkerFaceColor',[20 111 170] / 255,'Color',[20 111 170] / 255); % 使用'o'来表示数据点为圆圈形状
hold on;
p4 = plot(x, x, 'k--','linewidth', 1);
lg = legend([p3,p2,p1,p4],{'N', [fitted_line,', ',r2], '95% confidence interval','Parity line'},'location','NorthWest'); 
% 坐标区调整
axis tight
set(gca, 'Box', 'off', ...                                   % 边框
         'LineWidth', 1,...                                  % 线宽
         'XGrid', 'on', 'YGrid', 'on', ...                   % 网格
         'TickDir', 'out', 'TickLength', [.015 .015], ...    % 刻度
         'XMinorTick', 'off', 'YMinorTick', 'off', ...       % 小刻度
         'XColor', [.1 .1 .1],  'YColor', [.1 .1 .1])        % 坐标轴颜色
% 坐标轴刻度调整
set(gca, 'YTick', 0:100:500,...
         'Ylim' , [-10 510], ...
         'Xlim' , [-10 510], ...
         'XTick', 0:100:500) 
%'Xticklabel',{'S1' 'S2' 'S3' 'S4' 'S5' 'S6'}
% 字体和字号
set(gca, 'FontName', 'Arial', 'FontSize', 8)
set(lg, 'FontName',  'Arial', 'FontSize', 8)
hXLabel = xlabel('In situ emission rate (t CO_2/hr)');
hYLabel = ylabel('Estimated emission rate (t CO_2/hr)');
set([hXLabel,hYLabel], 'FontName',  'Arial', 'FontSize', 8)
% 背景颜色
set(gcf,'Color',[1 1 1])
% 添加上、右框线
xc = get(gca,'XColor');
yc = get(gca,'YColor');
unit = get(gca,'units');
ax = axes( 'Units', unit,...
           'Position',get(gca,'Position'),...
           'XAxisLocation','top',...
           'YAxisLocation','right',...
           'Color','none',...
           'XColor',xc,...
           'YColor',yc);
set(ax, 'linewidth',1,...
        'XTick', [],...
        'YTick', []);
%% 图片输出
fileout = 'test';
print(figureHandle,[fileout,'.png'],'-r300','-dpng');
```

效果如图

![picture1](/img/Matlab_Plotting_95CI/Matlab_figure.png "图2")


# 参考链接

[用成像光谱测量液化天然气（LNG）接收站的二氧化碳排放 - Zhang - 2023 - 地球物理研究快报 - Wiley Online Library](https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2023GL105755)

[Plotting different Confidence Intervals around Fitted Line using R and ggplot2](https://www.datawim.com/post/plotting-confidence-intervals-in-r/)

