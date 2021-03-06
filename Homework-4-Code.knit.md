---
title: "Homework 4"
author: "Ethan Hoffman, Gabrielle Barsotti, and Halley McVeigh"
date: "5/22/2021"
output:
  pdf_document: default
  html_document: default
---



### Question 1



```r
#Question 1
##Read in the damages.csv file and then mutate to set up the quadratic equation
damages <- read.csv(here("data", "damages.csv")) %>% 
  mutate(warm2 = warming^2)

##Create a linear model to find the estimated damage function in the damages data table
damages_lm <- lm(damages ~ warming + warm2, data = damages)
damages_lm[["coefficients"]][["(Intercept)"]] <- 0

##Create data visualization of damages.csv data
ggplot(data = damages, aes (x = warming, y = damages))+
  geom_point() +
  geom_smooth(data=damages, aes(x=warming, y=damages))+
  annotate(geom = "text", x = 3.5, y = 1000000000000000, label = "y = 1.95 * (10^-13) x^2 - 3.07 * (10^-12) x") +
  theme_minimal() +
  labs(x = "Warming (degrees C)", y = "Damages ($)", title = "Damages of climate change under different climate trajectories")
```

![](Homework-4-Code_files/figure-latex/unnamed-chunk-1-1.pdf)<!-- --> 

### Question 2


```r
#Question 2
##Read in the warming.csv file
warming <- read.csv(here("data", "warming.csv"))
```



```r
#Question 2
##Use the warming file and estimated damage function from Question 1 to create four plots
###Using the damages lm from Question 1, isolate the coefficient values to create this next graph.
x <- damages_lm[["coefficients"]][["warming"]]
x2 <- damages_lm[["coefficients"]][["warm2"]]
intercept <- damages_lm[["coefficients"]][["(Intercept)"]]

###Find the Difference in Damages over Time that Arise from the Pulse. After finding the difference, divide the difference by 35 billion tons of carbon which is roughly equal to the amount of annual global emissions
warming_difference <- warming %>%
  mutate(damage_base = (warming_baseline*x) + ((warming_baseline)^2*x2) + intercept,
         damage_high = ((warming_baseline*1.5)*x) + ((warming_baseline*1.5)^2*x2) + intercept,
         damage_pulse = (warming_pulse * x) + ((warming_pulse)^2 * x2) + intercept,
         damage_difference = damage_pulse - damage_base,
         damage_annual_cost = damage_difference/35000000000
         )
```

#### Plot 1-4

```r
###Damages Over Time without the Pulse
damage_plot <- ggplot(data = warming_difference, aes(x = year, y = damage_base))+
  geom_point()+
  theme_minimal() +
  labs(title = "Plot 1: Damages over time without pulse", x = "Year", y = "Damages ($)")

damage_plot
```

![](Homework-4-Code_files/figure-latex/unnamed-chunk-4-1.pdf)<!-- --> 

```r
###Damages Over Time with the Pulse
damage_pulse_plot <- ggplot(data = warming_difference, aes(x = year, y = damage_pulse))+
  geom_point()+
  theme_minimal() +
  labs(title = "Plot 2: Damages over time with pulse", x = "Year", y = "Damages ($)")

damage_pulse_plot
```

![](Homework-4-Code_files/figure-latex/unnamed-chunk-4-2.pdf)<!-- --> 

```r
###Damage difference Over Time without the Pulse
damage_difference_plot <- ggplot(data = warming_difference, aes(x = year, y = damage_difference))+
  geom_point()+
  theme_minimal() +
  labs(title = "Plot 3: Differences in damages over time from pulse", x = "Year", y = "Damages ($)")

damage_difference_plot
```

![](Homework-4-Code_files/figure-latex/unnamed-chunk-4-3.pdf)<!-- --> 

```r
###Damage difference over time per ton of carbon
damage_ton_plot <- ggplot(data = warming_difference, aes(x = year, y = damage_annual_cost)) +
  geom_point() +
  theme_minimal() +
  labs(title = "Plot 4: Difference in damages over time from pulse per ton of CO2", x = "Year", y = "Damages ($)")

damage_ton_plot
```

![](Homework-4-Code_files/figure-latex/unnamed-chunk-4-4.pdf)<!-- --> 

### Question 3



```r
#Question 3
##Create a plot of SCC vs. Discount Rate
warming_scc <- warming_difference

warming_scc <- warming_difference %>% 
  mutate(Discount1 = ((sum(warming_scc$damage_annual_cost))/(1+0.01)^80),
         Discount2 = ((sum(warming_scc$damage_annual_cost))/(1+0.02)^80),
         Discount3 = ((sum(warming_scc$damage_annual_cost))/(1+0.03)^80),
         Discount4 = ((sum(warming_scc$damage_annual_cost))/(1+0.04)^80),
         Discount5 = ((sum(warming_scc$damage_annual_cost))/(1+0.05)^80))

scc_table <- warming_scc %>% 
  select("Discount1", "Discount2", "Discount3", "Discount4", "Discount5") %>% 
  slice(1)

scc_table
```

```
##   Discount1 Discount2 Discount3 Discount4 Discount5
## 1  81.23011  36.93288  16.92189  7.811956  3.633147
```

```r
trans_scc_table <- t(scc_table)

discount_scc = data.table(
  SCC = c(81.2301056376792, 36.9328804024112, 16.9218929850311, 7.8119557882259, 3.6331472085021),
  Discount_rate = c(1, 2, 3, 4, 5)
)

discount_scc
```

```
##          SCC Discount_rate
## 1: 81.230106             1
## 2: 36.932880             2
## 3: 16.921893             3
## 4:  7.811956             4
## 5:  3.633147             5
```

```r
ggplot(data = discount_scc, aes(x = Discount_rate, y = SCC)) +
  geom_point() +
  geom_smooth(data = discount_scc, aes(x = Discount_rate, y = SCC), span = 1, se = FALSE) +
  labs(title = "Social Cost of Carbon (SCC) by discount rates", x = "Discount rates (%)", y = "Social Cost of Carbon (SCC)")
```

![](Homework-4-Code_files/figure-latex/unnamed-chunk-5-1.pdf)<!-- --> 

### Question 4 

#### The SCC with a discount rate of 2.1% is $34.15 calculated using the Ramsey Rule. This value is represented on the plot below, shown in coral.


```r
# Question 4 Ramsey Rule
p = .001
n = 2
g = 0.01

r = p + n*g

r
```

```
## [1] 0.021
```

```r
SCC2.1 = ((sum(warming_scc$damage_annual_cost))/(1+0.021)^80)
SCC2.1
```

```
## [1] 34.14818
```

```r
# Add 2.1% discount rate SCC point on plot
ggplot(data = discount_scc, aes(x = Discount_rate, y = SCC)) +
  geom_point() +
  geom_smooth(data = discount_scc, aes(x = Discount_rate, y = SCC), span = 1, se = FALSE) +
  labs(title = "Social Cost of Carbon (SCC) by discount rates", x = "Discount rates (%)", y = "Social Cost of Carbon (SCC)") +
  geom_point(aes(x = 2.1, y = 34.14818, color = "coral12"), show.legend = FALSE)
```

![](Homework-4-Code_files/figure-latex/unnamed-chunk-6-1.pdf)<!-- --> 

### Question 5

#### The expected present value of damages up to 2100 under Policy A is $1,869,052,186,623,438.

#### The expected present value of damages up to 2100 under Policy B is $1,422,733,591,966,428.

#### The difference between damages under Policy A and Policy B given X costs for Policy B is $446,318,594,657,010.

#### A risk-averse population would be inclined to favor Policy B because Policy A has a 50% probability that the damages will be greater than under Policy B. Despite Policy B having associated costs, unlike under Policy A, it still is more economically stable.


```r
# Question 5
## Expected present value of damages up to 2100 under Policy A 

p1 = 0.5
x_1 = sum(warming_difference$damage_base)/(1+0.02)^80
p2 = 0.5
x_2 = sum(warming_difference$damage_high)/(1+0.02)^80

E = (p1 * x_1) + (p2 * x_2)

E
```

```
## [1] 1.869052e+15
```

```r
## Expected present value of damages up to 2100 under Policy B
policy_b_2050 <- warming_difference %>% 
  filter(year < 2051)

a = sum(warming_difference$damage_base)
b = (2.853968e+13)*50
a + b
```

```
## [1] 6.936451e+15
```

```r
E_policy_b = (a + b)/(1+0.02)^80
E_policy_b
```

```
## [1] 1.422734e+15
```

```r
## Find the difference between damages under Policy A and Policy B given X costs for Policy B
X_cost = E - E_policy_b
X_cost
```

```
## [1] 4.463186e+14
```

```r
## A risk-averse population would be inclined to favor Policy B because Policy A has a 50% probability that the damages will be greater than under Policy B. Despite Policy B having associated costs, unlike under Policy A, it still is more economically stable.
```

