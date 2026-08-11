# Urban Heat Island Classifier — Deployment Package

This folder contains everything needed to deploy your K-Means capstone project
as a live web app, e.g. on an AWS EC2 instance.

## Files

| File | Purpose |
|---|---|
| `train_model.py` | Recreates the exact pipeline from your notebook (StandardScaler + KMeans, k=4) and saves **`model.pkl`** |
| `urbanheatisland_dataset_test.csv` | The dataset `train_model.py` needs to fit the model |
| `app.py` | Flask app — loads `model.pkl`, serves the form, and predicts a cluster from user input |
| `templates/index.html` | The interactive front-end (sliders, live result panel, no page reload) |
| `requirements.txt` | Python packages to install |

## How it works

Your notebook is **unsupervised clustering**, not a yes/no classifier — there is
no "placed / not placed" style label to predict. So instead of predicting a
class, the app:

1. Takes the same 13 features your notebook used (Latitude, Longitude,
   Elevation, Temperature, Population Density, Energy Consumption, AQI,
   Urban Greenness Ratio, Health Impact, Wind Speed, Humidity, Rainfall, GDP).
2. Scales them with the **same fitted `StandardScaler`**.
3. Assigns the input to one of the **4 KMeans clusters**.
4. Labels that cluster with whichever real `Land Cover` type was most common
   in it during training (e.g. Cluster 2 → "Urban"), so the result is
   readable instead of just showing "Cluster 2".
5. Shows a similarity score to all 4 clusters and compares your inputs to
   that cluster's feature averages.

## Step-by-step: build the pkl and run locally first

```bash
# 1. Create/activate a virtual environment (recommended)
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 2. Install dependencies
pip install -r requirements.txt

# 3. Generate model.pkl (run this ONCE — needs the csv in the same folder)
python train_model.py
# -> prints "Saved model.pkl" and a cluster summary

# 4. Run the app
python app.py
# -> open http://localhost:8080 in your browser
```

If you retrain in Jupyter/Colab instead, just make sure the notebook ends by
saving the same `model.pkl` structure `train_model.py` produces (scaler +
kmeans + feature_cols + cluster_profiles + feature_ranges in one dict), then
download that `model.pkl` into this folder.

## Deploying on AWS (EC2)

1. Launch an EC2 instance (Ubuntu, t2.micro is enough for a demo).
2. Open port **8080** (or 80) in the instance's Security Group — same idea
   your `app.py` already uses (`app.run(host='0.0.0.0', port=8080)`).
3. Upload this whole folder to the instance (`scp` or via GitHub).
4. SSH in, then:
   ```bash
   sudo apt update && sudo apt install -y python3-pip
   pip3 install -r requirements.txt
   python3 train_model.py      # only if model.pkl isn't already uploaded
   python3 app.py
   ```
5. Visit `http://<EC2-public-IP>:8080` in your browser.

For a persistent/production setup later, run `app.py` under `gunicorn` behind
`nginx` instead of the Flask dev server — but for a capstone demo, the steps
above are exactly the "pkl + app.py + templates" flow your instructor
described, and are enough to get it running end-to-end without errors.

## Common errors this avoids

- **Feature order mismatch** — `app.py` always builds the input array in the
  exact `feature_cols` order stored inside `model.pkl`, so it can't silently
  swap two columns.
- **Unscaled input** — the same fitted `scaler` from training is reused, not
  a fresh one.
- **`UnicodeDecodeError`** — the pkl bundle stores the feature names natively
  in Python (not written back out to a CSV), so the `°C` / `km²` characters
  don't get mangled the way they did in your earlier Colab file-reading issue.
