# How do I even get started with Data Visualization?
**[American Geophysical Union](https://www.agu.org/) in partnership with [Don’t Use This Code](https://dutc.io)**

*Wednesday, July 12, 2023 at 2pm US/Eastern*

Welcome to the workshop: “How do I even get started with Data Visualization”

In order to follow along with today’s presentation we have provided 2 options for you:

## 1. Google Colab

Google Colab is an online notebook-style editor for you to interactively run Python code in.
In order to get started with Google Colab, you may use the below link:

*Note* that you will need to sign into your Google Account.

https://colab.research.google.com/github/dutc-io/agu-data-viz/blob/main/notes.ipynb

Make sure the execute the cells under the **If Running From Google Colab** Section! (This will prompt you to sign into your Google Account if you are not already)

**If you want to SAVE your notes/code from today, make sure to download your notebook from Google Colab!**

## 2. Git & Anaconda (advanced)

For the purposes of this workshop, using Google Colab will be faster. I would
only advocate for this approach if you are familiar with these tools.

If you have working knowledge of `git` and `conda`, then you can locally clone the github repository and
create a conda virtual environment from the `environment.yml` file.

Start a new Terminal session and run the following commands

*Note: The environment will take a few minutes to resolve!*

```bash
git clone https://github.com/dutc-io/agu-data-viz.git
cd agu-data-viz
conda env create -f environment.yml  # this step can take a while!
conda activate agu-data-viz-02
jupyter lab
```

To open the jupyter lab interface in your browser, copy/paste `http://localhost:...` link
from your terminal’s output into your browser and hit <kbd>Enter</kbd>.


