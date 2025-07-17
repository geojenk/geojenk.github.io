### Updated ML Model for Breast Cancer Tumor Diagnosis

View the full project on [GitHub](https://github.com/geojenk/BreastCancerMachineLearning/tree/main).

<img src="images/tumor_cells.jpg?raw=true"/>

At the time of the report by Street, Wolberg, and Mangasarian (1992), breast masses  were traditionally diagnosed by full biopsy, an invasive surgical procedure. Fine needle aspiration (FNA) allows doctors to take a much smaller tissue sample, but diagnosis based on an FNA sample was difficult because of the high number of nuclear features thought to be correlated with a malignant tumor. Street et. Al chose 3 features from the 10 obtained through FNA and developed a multi-surface method tree (MSM-Tree) model to classify the masses as malignant or benign. They achieved 97.3% accuracy, 90% sensitivity, and 86% specificity.  Although this model’s performance was strong, there is value in improving the sensitivity and specificity given the life-or-death, emotional, and financial implications of the results. 

With advent of random forest models, which were first described in 1995, there is potential to correctly classify tumors with even higher accuracy, sensitivity, and specificity. This project saught to do just that, building a Random Forest model with 98.25% accuracy, 97.62% sensitivity, and 97.62% specificity. 
