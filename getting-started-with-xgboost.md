## Introduction

Recently I have been enjoying the machine learning tutorials on [Kaggle.com](https://www.kaggle.com/richardhildebrand).

While their environment is very nice, I still prefer to do much of my work locally, so I wanted to setup my local machine to crunch csv files with tools like `Pandas` and `XGBRegressor`.

## My first attempt

To get started, I decided to use the code provided in [Dan Becker's XGBoost tutorial](https://www.kaggle.com/dansbecker/learning-to-use-xgboost/notebook)

[code="python"]

    import pandas as pd
    from sklearn.model_selection import train_test_split
    from sklearn.preprocessing import Imputer

    data = pd.read_csv('data/house_prices/input/train.csv')
    data.dropna(axis=0, subset=['SalePrice'], inplace=True)
    y = data.SalePrice
    X = data.drop(['SalePrice'], axis=1).select_dtypes(exclude=['object'])
    train_X, test_X, train_y, test_y = train_test_split(X.as_matrix(), y.as_matrix(), test_size=0.25)

    my_imputer = Imputer()
    train_X = my_imputer.fit_transform(train_X)
    test_X = my_imputer.transform(test_X)

    from xgboost import XGBRegressor

    my_model = XGBRegressor()
    # Add silent=True to avoid printing out updates with each cycle
    my_model.fit(train_X, train_y, verbose=False)

    predictions = my_model.predict(test_X)

    from sklearn.metrics import mean_absolute_error
    print("Mean Absolute Error : " + str(mean_absolute_error(predictions, test_y)))
    
[/code]

Unfortunately I received the following error: 

[code="python"]

	ModuleNotFoundError: No module named 'pandas'
[/code]

It looks like we will need to install a few modules first.

## Installing pandas

[code="python"]

    pip install wheel
    pip install pandas
[/code]

This gets us past our pandas issue, but leaves us in a similar state for sklearn:

[code="python"]
    ModuleNotFoundError: No module named 'sklearn'
[/code] 

## Installing sklearn

Much like pandas, sklearn has a few dependencies, so lets install them all at once.

[code="python"]

    pip install -U numpy scipy scikit-learn
[/code]

Once again we get to move forward, but are now stuck when importing from `xgboost` with the error:

[code="python"]

    ModuleNotFoundError: No module named 'xgboost'
[/code]

## Installing xgboost

Now we might expect that `pip install xgboost` would solve our problem the same way it has before. Unfotrunetly this yields the following error: 

[code="python"]

    No files/directories in C:\Users\rhildebr\AppData\Local\Temp\pip-install-blv0yelu\xgboost\pip-egg-info (from PKG-INFO)`.
  
[/code]

It looks like this one will be a bit more trickey.

Since `pip` won't help us here, lets download the appropriate file from [https://www.lfd.uci.edu/~gohlke/pythonlibs](https://www.lfd.uci.edu/~gohlke/pythonlibs/#xgboost).

Now we can navigate to our downloads folder and run: 

[code="python"]

    pip install file_we_downloaded.whl
[/code] 

I had to try a few files before I was able to find the correct one for my system. The file that worked for me was `xgboost-0.71-cp36-cp36m-win32.whl` and it yielded the success message `Successfully installed xgboost-0.71`.

## All done

Our tutorial code finally runs, outputting our `Mean Absolute Error`!
