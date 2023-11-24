# Machine_Learning_Practice_Tourette_Syndrome

## Reference
These data are collected by Yu-Jiun Chen in National Taiwan University Hospital Hsin-Chu Branch.  
For the detail of this experiment, please read Yu-Jiun Chen's master thesis.

---

## Content
1. About
2. Data Information 
3. Code Information
    1. Data Preprocess
    2. Feature Extraction
    3. Machine Learning

---
## About
- In this study, we apply neurofeedback training  based on functional near-infrared spectroscopy as the behavior therapy for TS patients, and last for 8 weeks.  

- Yale global tic severity scale (YGTSS) is applyed in the first and last week to evaluate the improvment of patients, the mild group and severe group is bounded by score of 25.

- The classification machine leaning model helps us the figure out the differences between mild group and severe group.

## Data Information

### Data Information of Patients
The anonymous patient list is saved in `patient_list.csv`

| Column name | Details |
| --- | --- |
| Case | Case number|
| Age | Age
| Pre-test | The YGTSS score\* in first week |
| Post-test | The YGTSS score in eighth week |

\* Severe and mild group are bounded by 25. 

### Data information for NIRS Files
The files are placed by the case number, `usually` two files for each case. Check it by yourself.

\#Useful columns

| Column name | Details |
| --- | --- |
| Time_Host | Time |
| CH1_PD730 | The intensity in ch1 at 730nm, range in 0 to 4095 |
| CH1_PD850 | The intensity in ch1 at 850nm, range in 0 to 4095 |
| CH2_PD730 | The intensity in ch2 at 730nm, range in 0 to 4095 |
| CH2_PD850 | The intensity in ch2 at 850nm, range in 0 to 4095 |
| CH3_PD730 | The intensity in ch3 at 730nm, range in 0 to 4095 |
| CH3_PD850 | The intensity in ch3 at 850nm, range in 0 to 4095 |
| CH4_PD730 | The intensity in ch4 at 730nm, range in 0 to 4095 |
| CH4_PD850 | The intensity in ch4 at 850nm, range in 0 to 4095 |
| trail_times | Number of trails, 32 in total |

---

## Code Information
### Data Preprocess
![Flow Chart for Data Preprocess](./figs/Data_preprocess.png "Data_preprocess")
        
### Feature Extraction
- 7 features are extracted in this file:
1. stage average:  
    The average value in each stage.
2. average difference:  
    The difference of average value between two stages.
3. stage peak:  
    The maximum value in each stage.
4. stage activation:  
    The difference between maximum - minimum value in each stage.
5. begin slope (trail, rest):  
    The slope in the begining of trail & rest stage.
6. standard deviation:  
    The standard deviation value in each stage.
7. standard deviation difference:  
    The difference of standard deviation value between two stages.

### Machine Learning
1. Data preprocess:  
   - Use ```train_test_split``` to make training and test data.
2. Statistics:
   - Check if the variables are significant different in two groups
   - Steps:  
     1. Shapiro test: Test if samples have normal distribution
     2. Levene test: Test if samples have equal variances
     3. Two-tailed t-test: Test both if the mean is significantly greater than x and if the mean significantly less than x
```python
good_feature = []
p_values = []
for feature in XH_train.columns:
    sat, p_value_H = shapiro(XH_train[feature])
    sat, p_value_L = shapiro(XL_train[feature])
    if p_value_H>0.05 and p_value_L>0.05:
        sat, p_value = levene(XH_train[feature], XL_train[feature])
        if p_value > 0.05:
            sat, p_value = ttest_ind(XH_train[feature], XL_train[feature], equal_var = True)
            if p_value <0.05:
                good_feature.append(feature)
                p_values.append(p_value)
```
3. SVM:
   - Use `StandardScaler` to standardize data.
   - Use `ParameterSampler` to find best paramenter.
   - Fit and predict with `SVC`.
```python
stdd = StandardScaler()
X_train_std = stdd.fit_transform(X_train)
X_test_std = stdd.transform(X_test)

param_grid={"gamma":[0.001, 0.01, 0.1, 1], "C":[0.01, 0.1, 1, 10, 100]}
pam = ParameterSampler(param_grid, n_iter=20)
for parameter in list(pam):
    svc = SVC(kernel='rbf', **parameter)

    svc = svc.fit(X_train_std, y_train)
    y_predict = svc.predict(X_test_std)
    y_predict_train = svc.predict(X_train_std)

    if accuracy_score(y_train, y_predict_train)>0.8 and accuracy_score(y_test, y_predict)>0.70:
        param_list.append(str(parameter))
        train_acc.append(accuracy_score(y_train, y_predict_train))
        test_acc.append(accuracy_score(y_test, y_predict))
```
4. Plots:
    - Visulize the result.
5. Confusion matrix & box plot:
    - Use `confusion_matrix` to evaluate the predictions.
    - Use `boxplot` to figure out the distributions of varuables in two groups.
