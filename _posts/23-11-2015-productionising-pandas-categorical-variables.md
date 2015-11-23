---
title: Productionising a model using Pandas Categorical variables.
layout: post
---

One of the major limitations of Scikit Learn is its lack of native support of categorical variables for tree models. One way around this is to use `pd.get_dummies` to perform a one-hot-encoding on categorical variables, however this doesn't work well when the column has a lot of levels. Instead it can actually be more effective to convert the column to a numeric id and treat it as a continuous variable – as long as your trees have enough depth, they will naturally re-segment the variable into its natural categories. (I have routinely achieved up to 5% improvement on AUROC using this techique compared to a model of the same size/complexity using dummy variables.) The introduction of the `categorical` data type in Pandas since version 0.15 makes this a breeze -- the column is stored as a vector of Numpy `int64`, together with a list of levels which is used as a codebook.

I've been trying to work out the a natural way to save these encodings for productionising a model. I've settled on pickling an empty dataframe with the categorical predictor columns. Then when I append the data that is to be scored against the model, the categorical encoding is automatically applied. As a bonus, if a new value is encountered which is not present in the original dataframe, it will simply be set as `NaN`, which is exactly what I would hope for (this is a bit trickier to deal with when using dummy variables).

Here is a toy example to show what I mean

```python
import pandas as pd
import pickle

df = pd.DataFrame({'happy':[1,1,0],
                   'weather':['sunny','cloudy','cloudy'],
                   'snack':['fruit','cake','fruit']})\
       .apply(lambda x: x.astype('category'))

model = {(0,0): 1, (1,0): 0, (0,0): 1, (0,1):0}

#create template as a slice consisting of no rows, and columns 1 and 2
input_template = df.iloc[[], [1,2]]

model_and_input_template = {'model': model, 'input_template': input_template}
with open('model.pkl', 'wb') as f:
    f.write(pickle.dumps(model_and_input_template))

def score_data(input_data, model, input_template):
    with open('model.pkl', 'rb') as f:
        model_and_input_template = pickle.loads(f.read())
    model = model_and_input_template['model']
    input_template = model_and_input_template['input_template']    
    categorical_data = input_template.append(input_data, ignore_index=True)
    encoded_data = categorical_data.apply(lambda x : x.cat.codes if x.dtype == 'category' else x)
    return model[tuple(encoded_data.ix[0].values)]

input_data = {'weather':'cloudy','snack':'cake'}
print (score_data(input_data, model, input_template))
#1
#it's cloudy outside but the birthday cake must have cheered me up!
```
