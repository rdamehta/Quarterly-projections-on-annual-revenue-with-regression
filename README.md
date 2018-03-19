# Iowa Liquor Sales + Linear Regression

### Overview
I wanted to use scikit-learn to create linear regression models, disoover how goals & business objectives translate to model fit, and how to optimize models using cross-validation.

The state of Iowa provides many datasets on their website, including [this](https://www.dropbox.com/s/5oiz27mhvsiibo8/iowa_liquor_sales_proj_2.csv?dl=0) which contains transactions for all stores that have a class E liquor license.

They also offer [this 10% dataset version of Iowa liquor sales](https://drive.google.com/file/d/0Bx2SHQGVqWaseDB4QU9ZSVFDY2M/view?usp=sharing). I actually built my model on this data set first, and then scaled it to the larger dataset


**Goals:** 

* Calculate the yearly liquor sales for each store using the provided data. Adding up the transactions for each year, and store sales in 2015 which I used as my target variable
* Use the data from 2015 to make a linear model using as many variables as I found useful to predict the yearly sales of all stores. I used the sales from January to March as one of my variables
* estimated total sales in 2016, extrapolating from the sales so far for January to March of 2016.
* Used cross-validation to check how my  model predicts to held out data compared to the model metrics on the full dataset.
* Fit my model(s) using ridge regression and comparted to my non-regularized model


### Useful Resources
- [Documentation for SKLearn](http://scikit-learn.org/stable/user_guide.html)
- [What is regularization?](https://www.quora.com/What-is-regularization-in-machine-learning)
- [Summary on what an executive summary entails](http://www.umuc.edu/current-students/learning-resources/writing-center/writing-resources/executive-summaries/index.cfm?noprint=true)
- [How to write an executive summary for a proposal](https://www.proposify.biz/blog/executive-summary)
- [An example of financial and marketing executive summaries](https://unilearning.uow.edu.au/report/4bi1.html)

---

