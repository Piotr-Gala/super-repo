# Lesson 7: ML Data, Preprocessing, and Model Choice

Goal of this lesson: you should be able to explain **what data was used**, **how it was prepared**, and **why Random Forest was selected**.

## 1. Dataset overview

AeroSense used two complementary data sources:

- an IoT dataset with **Normal** and **Cooking** examples,
- the Blattmann smoke-sensor dataset with **Normal** and **Fire** examples.

After preprocessing, the merged dataset had:

```text
60,212 readings total
40,823 Fire    (67.8%)
14,671 Normal  (24.4%)
4,718 Cooking  (7.8%)
```

Important:

> The dataset is imbalanced. Fire is the majority class, and Cooking is the smallest class.

Good answer:

> We combined datasets because our own IoT data covered normal and cooking situations, while the external smoke dataset provided fire examples. This allowed us to train a three-class classifier, but it also introduced limitations because the data came from different sensor setups.

## 2. Why Cooking is its own class

Cooking is the special part of the project.

Cooking can increase:

- temperature,
- humidity,
- TVOC,
- eCO2.

This can look suspicious, but it should not automatically create a fire alarm.

Good answer:

> Cooking was kept as a separate class because the system should distinguish between dangerous smoke and normal cooking activity. This reduces unnecessary fire alarms and makes the classification more useful for residents.

## 3. Data inspection

Before training, the team checked:

- column names,
- data types,
- missing values,
- duplicate readings,
- feature ranges,
- class counts,
- source differences.

The cleaned final table had no missing values.

The report also mentions useful patterns:

- TVOC helped identify Fire,
- temperature helped separate Cooking,
- no single feature was enough as a complete rule.

Good answer:

> The exploration showed that different features contributed different signals. TVOC was strong for fire-like readings, temperature helped with cooking, and eCO2 added extra information. Because no single measurement was reliable enough alone, a model using multiple features was more appropriate than a simple threshold rule.

## 4. Cleaning

For IoT data, the team removed readings that looked like sensor warm-up or communication errors:

- TVOC equal to zero,
- eCO2 equal to zero,
- negative CO2,
- duplicates,
- temperature/humidity outliers using IQR.

For Blattmann data, the team:

- removed unavailable metadata columns,
- aligned names with the IoT schema,
- removed impossible values,
- removed duplicates,
- removed temperature/humidity outliers.

Trade-off:

> High gas values were kept because they may be real signs of cooking or smoke. Removing them as "outliers" would damage the actual signal we need.

## 5. Feature selection

The final feature vector is:

```text
temperature
humidity
tvoc
eco2
```

Why not `co2Level` and `aqi`?

Because they were not consistently available in both data sources. Keeping them would make the merged training dataset inconsistent with the model goal.

Good answer:

> We selected only the features that existed in both the prepared datasets and the deployed sensor payload. This avoided training a model on fields that would not be available consistently at prediction time.

## 6. Scaling and leakage

The model pipeline used `StandardScaler`.

Why?

Features have different units and scales:

- temperature is around tens,
- humidity is percent,
- TVOC and eCO2 can be hundreds or thousands.

Scaling prevents large numeric ranges from dominating algorithms such as KNN or SVM.

Important detail:

> The scaler was fitted on the training set only, then applied to validation and test sets. This avoids preprocessing leakage.

The fitted scaler was saved as:

```text
System/MlServer/model/scaler.joblib
```

## 7. Train/validation/test split

The report describes a reproducible stratified split:

```text
70% training
15% validation
15% test
random_state = 42
```

Why stratified?

Because the dataset is imbalanced. Stratification keeps similar class proportions in train, validation, and test sets.

Good answer:

> Stratified splitting was important because Cooking was the smallest class. Without stratification, validation or test results could become misleading if one split had too few Cooking samples.

## 8. Candidate models

The team compared several candidates:

- KNN,
- SVM,
- Random Forest,
- Gradient Boosting,
- XGBoost.

Simple explanation:

- KNN compares a reading to nearby historical readings, but depends heavily on scaling.
- SVM finds decision boundaries, but also needs careful preprocessing.
- Random Forest combines many decision trees and is robust for tabular data.
- Gradient Boosting builds trees sequentially and can perform very well.
- XGBoost is a faster/optimized boosting approach.

## 9. Evaluation priority

The main evaluation priority was **Fire recall**.

Why?

Because the worst error is:

```text
actual Fire -> predicted Normal
```

That means the system misses a dangerous event.

Second priority:

```text
actual Cooking -> predicted Fire
```

This creates false fire alarms during cooking.

Good answer:

> We did not look only at accuracy. In a fire-risk system, missing a real fire is more dangerous than a normal classification error, so Fire recall mattered most. After that, we cared about reducing Cooking-to-Fire mistakes because avoiding false cooking alarms was one of the main project goals.

## 10. Final model choice

The final deployed model is Random Forest:

```text
System/MlServer/model/model_random_forest.joblib
```

Why Random Forest?

The report says Random Forest and Gradient Boosting performed similarly, but Random Forest performed best on the test dataset and was chosen for deployment.

Good answer:

> Random Forest was selected because it performed best among the final candidates on the prepared test data, handled the tabular sensor features well, and gave stable predictions for the three target classes. It was also practical to save and deploy through `joblib` in the FastAPI service.

## Limitations

Be honest. This is important.

Main limitations:

- Fire examples came mostly from an external dataset, not from the deployed AeroSense hardware.
- The merged dataset is imbalanced, with Fire as the majority class.
- The model has not been validated in a real building environment.
- Room geometry, ventilation, sensor placement, and different hardware can change readings.
- Test-set performance is not a guarantee of real-world safety.

Good answer:

> The model is suitable as a prototype classifier, but it should not be treated as production safety certification. The biggest limitation is that the training data combines different sources, and the fire examples do not come from the exact deployed AeroSense hardware in real building conditions.

## Best exam answer

> The ML dataset combined IoT readings for Normal and Cooking with the Blattmann smoke dataset for Fire examples. After cleaning and merging, the model used four shared features: temperature, humidity, TVOC, and eCO2. The task was treated as three-class classification because Cooking should not automatically trigger a fire alarm. The data was split with a stratified 70/15/15 train, validation, and test strategy to handle class imbalance. We compared models such as KNN, SVM, Random Forest, and Gradient Boosting, and selected Random Forest because it performed best on the prepared test data. The most important metric concern was Fire recall, because missing a real fire is the most dangerous error.

## Remember

```text
Two data sources
+ three classes
+ four shared features
+ stratified split
+ Fire recall matters most
+ Random Forest deployed
+ real-world validation still missing
```
