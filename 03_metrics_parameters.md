# DVC Tutorial: Part 3 - Metrics and Parameters

This is where DVC elevates your workflow from simply "saving files" to actively
managing machine learning experiments.

When you are tuning a model, you change hyperparameters constantly. If you
don't track *what* parameters produced *which* accuracy, you'll quickly lose
track of your best model. DVC solves this by natively tracking parameters and
metrics, allowing you to compare them across Git commits.

## The Goal

Instead of hardcoding the Random Forest configuration (like `n_estimators`)
directly inside our Python script, we will extract it into a configuration
file. Then, we will update our script to calculate the model's accuracy and
save it to a metrics file. Finally, we will use DVC to compare different runs.

## 1. Environment Update

To easily read YAML configuration files in Python, we need one additional
package. Choose your package manager to install it:

```sh
# uv
uv add pyyaml

# pip
pip install pyyaml

# conda
conda install conda-forge::pyyaml

```

## 2. Creating the Parameters File

DVC automatically looks for a file named `params.yaml` by default. Let's create
it in the root of your repository:

```yaml
# params.yaml
train:
  n_estimators: 100
  random_state: 42

```

## 3. Updating the Training Script

Now, let's update `train.py` so that it dynamically reads the parameters from
`params.yaml` and calculates a simple accuracy score, saving it to
`metrics.json`.

Overwrite your `train.py` with this updated code:

```python
# train.py
import pandas as pd
import yaml
import json
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score
import pickle

print("Loading params...")
with open("params.yaml", "r") as f:
    params = yaml.safe_load(f)["train"]

print("Loading data...")
df = pd.read_csv('data/data.csv')
X = df[['feature_1', 'feature_2']]
y = df['label']

print("Training model...")
clf = RandomForestClassifier(
    n_estimators=params['n_estimators'], 
    random_state=params['random_state']
)
clf.fit(X, y)

print("Calculating metrics...")
# In a real scenario, you would evaluate on a separate test set!
predictions = clf.predict(X)
accuracy = accuracy_score(y, predictions)

with open("metrics.json", "w") as f:
    json.dump({"accuracy": accuracy}, f)

print("Saving model...")
with open('model.pkl', 'wb') as f:
    pickle.dump(clf, f)
    
print(f"Success! Accuracy: {accuracy}")

```

## 4. Updating the Pipeline Definition

We need to tell DVC about these new files. We could run a `dvc stage
add` command again, but since we already have a `dvc.yaml` file from Part 2,
it's much easier to just edit it directly!

Open `dvc.yaml` and update it to look exactly like this:

```yaml
stages:
  train:
    cmd: python train.py
    deps:
    - data/data.csv
    - train.py
    params:
    - train.n_estimators
    - train.random_state
    outs:
    - model.pkl
    metrics:
    - metrics.json:
        cache: false

```

Notice the two new sections:

* **`params:`** Tells DVC to watch specific keys inside `params.yaml`. If these
values change, DVC knows the pipeline is out of date.
* **`metrics:`** Tells DVC that `metrics.json` is a special output. Setting
`cache: false` means Git will track this lightweight JSON file directly, rather
than storing it in DVC's heavy cache.

## 5. Running and Tracking the Baseline

Let's run our newly updated pipeline to establish our baseline:

```sh
dvc repro

```

You should see DVC execute the script and output the accuracy. Now, let's
commit this baseline to Git:

```sh
git add train.py params.yaml dvc.yaml dvc.lock metrics.json
git commit -m "Add parameters and metrics tracking (Baseline)"

```

## 6. Running an Experiment

Let's see if a smaller Random Forest performs differently.

You can either edit the `params.yaml` file or set the
parameter for the experiment live with:

Run the pipeline again:

```sh
dvc exp run -S train.n_estimators=3
```

Because you defined `n_estimators` as a dependency in `dvc.yaml`, DVC instantly
detects the change and reruns the `train` stage.

## 7. Comparing Experiments

This is where the magic happens. You now have two different commits in Git with
two different model configurations. You can compare them directly in your
terminal without digging through code.

To compare the parameters between your current workspace and the previous commit, run:

```sh
dvc params diff

```

You will see a clean table showing exactly what changed (`n_estimators` going
from 100 to 10).

To compare the resulting performance metrics, run:

```sh
dvc metrics diff

```

This will output a comparison table showing the accuracy difference between the
two runs. You instantly know if your parameter change improved or degraded the
model!
