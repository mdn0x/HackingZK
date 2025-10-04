# Environment Setup

## Miniconda

`Miniconda` is a minimal installer for the `Anaconda` distribution of the [Python](../../../3%20-%20Tags/Programming%20Languages/Python.md) programming language. It provides the `conda` package manager and a core Python environment without automatically installing the full suite of data science libraries available in `Anaconda`. Users can selectively install additional packages, creating a customized environment that aligns with their specific needs.

Both `Miniconda` and `Anaconda` rely on the `conda` package manager, allowing for simplified installation, updating, and management of Python packages and their dependencies. In essence, `Miniconda` offers a lighter starting point, while `Anaconda` comes pre-loaded with a broader range of commonly used data science tools.
### Why Miniconda?

You might wonder why we use `Miniconda` instead of a standard `Python` installation. Here are a few compelling reasons:

- `Performance:` `Miniconda` often performs data science and machine learning tasks better due to optimized packages and libraries.
- `Package Management:` The `Conda` package manager simplifies package installation and management, ensuring compatibility and resolving dependencies. This is particularly crucial in deep learning, where projects often rely on a complex web of interconnected libraries.
- `Environment Isolation:` `Miniconda` allows you to create isolated environments for different projects. This prevents conflicts between packages and ensures each project has its dedicated dependencies.

By using `Miniconda`, you'll streamline your workflow, avoid compatibility issues, and ensure that your deep learning environment is optimized for performance and efficiency.

### Linux installation

Miniconda provides a straightforward installation process that relies not solely on a distribution’s package manager. You can obtain the latest Miniconda installer directly from the official repository, run it silently, and then load the conda environment for your user shell. This approach ensures that conda commands and environments are readily available without manual configuration.

```shell-session
mdn0x@htb[/htb]$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
mdn0x@htb[/htb]$ chmod +x Miniconda3-latest-Linux-x86_64.sh
mdn0x@htb[/htb]$ ./Miniconda3-latest-Linux-x86_64.sh -b -u
mdn0x@htb[/htb]$ eval "$(/home/$USER/miniconda3/bin/conda shell.$(ps -p $$ -o comm=) hook)"
```

Tips: add `eval "$(/home/$USER/miniconda3/bin/conda shell.$(ps -p $$ -o comm=) hook)"` to the `.zshrc` or similar file.

Confirm that Miniconda is installed correctly by running:

```shell-session
mdn0x@htb[/htb]$ conda --version

conda 25.7.0
```
### Init

The `init` command configures your shell to recognize and utilize `conda`. This step is essential for:

- `Activating environments:` Allows you to use `conda activate` to switch between environments.
- `Using conda commands:` Ensures that `conda` commands are available in your shell.

To initialize `conda` for your shell, run the following command after installing `Miniconda`:

Environment Setup

```shell-session
mdn0x@htb[/htb]$ conda init
```

This command will modify your shell configuration files (e.g., `.bashrc` or `.zshrc`) to include the necessary `conda` settings. You might need to close and reopen your terminal for the changes to take effect.

Finally, run these two commands to complete the `init` process

Environment Setup

```shell-session
mdn0x@htb[/htb]$ conda config --add channels defaults
mdn0x@htb[/htb]$ conda config --add channels conda-forge
mdn0x@htb[/htb]$ conda config --add channels nvidia # only needed if you are on a PC that has a nvidia gpu
mdn0x@htb[/htb]$ conda config --add channels pytorch
mdn0x@htb[/htb]$ conda config --set channel_priority strict
```
### Deactivating Base

After installing `Miniconda`, you'll notice that the `base` environment is activated by default every time you open a new terminal. This is indicated by the `(base)` prefix on your path.

Environment Setup

```shell-session
(base) $ 
```

While this can be useful, it's often preferable to start with a clean slate and activate environments only when needed. Personally, I wouldn't say I like seeing the `(base)` prefix all the time, either.

To prevent the `base` environment from activating automatically, you can use the following command:

Environment Setup

```shell-session
mdn0x@htb[/htb]$ conda config --set auto_activate_base false
```

This command modifies the `condarc` configuration file and disables the automatic activation of the `base` environment.

When you open a new terminal, you won't see the `(base)` prefix in your prompt anymore.
### Managing Virtual Environments

In software development, managing dependencies can quickly become a complex task, especially when working on multiple projects with different library requirements.

This is where `virtual environments` come into play. A virtual environment is an isolated space where you can install packages and dependencies specific to a particular project, without interfering with other projects or your system's global Python installation.

They are critical for AI tasks for a few reasons:

- `Dependency Isolation:` Each project can have its own set of dependencies, even if they conflict with those of other projects.
- `Clean Project Structure:` Keeps your project directory clean and organized by containing all dependencies within the environment.
- `Reproducibility:` Ensures that your project can be easily reproduced on different systems with the same dependencies.
- `System Stability:` Prevents conflicts with your global Python installation and avoids breaking other projects.

`conda` provides a simple way to create virtual environments. For example, to create a new environment named `ai` with Python 3.11, use the following command:

```shell-session
mdn0x@htb[/htb]$ conda create -n ai python=3.11
```

This will create a virtual environment, `ai`, which can then be used to contain all ai-related packages.
### Activating the Environment

To activate the `myenv` environment, use:

Environment Setup

```shell-session
mdn0x@htb[/htb]$ conda activate ai
```

You'll notice that your terminal prompt now includes the environment name in parentheses `(ai)`, indicating that the environment is active. Any packages you install using `conda` or `pip` will now be installed within this environment.

To deactivate the environment, use:

Environment Setup

```shell-session
mdn0x@htb[/htb]$ conda deactivate
```

The environment name will disappear from your prompt, and you'll be back to your base Python environment.
### Essential Setup

While `conda` provides a broad range of packages through its curated channels, it may not include every tool you require. In such cases, you can still use `pip` within the `conda` environment. This approach ensures you can install any additional packages that `conda` does not cover.

Use the `conda install` command to install the following core packages:

Environment Setup

```shell-session
mdn0x@htb[/htb]$ conda install -y numpy scipy pandas scikit-learn matplotlib seaborn transformers datasets tokenizers accelerate evaluate optimum huggingface_hub nltk category_encoders
mdn0x@htb[/htb]$ conda install -y pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia
mdn0x@htb[/htb]$ pip install requests requests_toolbelt
```
### Updates

`conda` provides a method to keep conda-managed packages up to date. Running the following command updates all conda-installed packages within the `(ai)` environment, but it does not update packages installed with `pip`. Any pip-installed packages must be managed separately, and mixing `pip` and `conda` installations may increase the risk of dependency conflicts.

Environment Setup

```shell-session
mdn0x@htb[/htb]$ conda update --all
```

---

# # JupyterLab


`JupyterLab` is an interactive development environment that provides web-based coding, data, and visualization interfaces. Due to its flexibility and interactive features, it's a popular choice for data scientists and machine learning practitioners.

## Why JupyterLab?

- `Interactive Environment`: `JupyterLab` allows running code in individual cells, facilitating experimentation and iterative development.
- `Data Exploration and Visualization`: It integrates seamlessly with libraries like `matplotlib` and `seaborn` for creating visualizations and exploring data.
- `Documentation and Sharing`: `JupyterLab` supports markdown and LaTeX for creating rich documentation and sharing code with others.

`JupyterLab` can be easily installed using `conda`, if it isn't already installed:

```shell-session
[!bash!]$ conda install -y jupyter jupyterlab notebook ipykernel 
```

Make sure you are running the command from within your ai environment.

To start `JupyterLab`, simply run:

```shell-session
[!bash!]$ jupyter lab
```

This will open a new tab in your web browser with the `JupyterLab` interface.

##  Using JupyterLab

`JupyterLab`'s primary component is the notebook, which allows combining code, text, and visualizations in a single document. Notebooks are organized into cells, where each cell can contain either code or markdown text.

- `Code cells`: Execute code in various languages (Python, R, Julia).
- `Markdown cells`: Create formatted text, equations, and images using markdown syntax.
- `Raw cells`: Untyped raw text.

Click the "Python 3" icon under the "Notebook" section in the Launcher interface to create a new notebook. This will open a notebook with a single empty code cell.

Type your Python code into the code cell and press `Shift + Enter` to execute it. For example:

```python
print("Hello, JupyterLab!")
```

The output of the code will appear below the cell.

`Jupyter` notebooks use a `stateful` environment, which means that variables, functions, and imports defined in one cell remain available to all later cells. Once you execute a cell, any changes it makes to the environment, such as assigning new variables or redefining functions, persist as long as the kernel is running. This differs from a `stateless` model, where each code execution is isolated and does not retain information from previous executions.

Being aware of the `stateful` nature of a notebook is important. For example, if you execute cells out of order, you might observe unexpected results due to previously defined or modified variables. Similarly, re-importing modules or updating variable values affects subsequent cell executions, but not those that were previously run.

Say you have a cell that does this:

```python
x = 1
```

then in a later cell you might have:

```python
print(x)  # This will print '1' because 'x' was defined previously.
```

If you change the first cell to:

```python
x = 2
```

and re-run it before running the `print(x)` cell, the value of `x` in the environment becomes `2`, so the output will now be different when you run the print cell.

Click the "+" button in the toolbar to add new cells. You can choose between code cells and markdown cells using the Dropdown on the toolbar. Markdown cells allow you to write formatted text and include headings, lists, and links.

`JupyterLab` integrates with libraries like `pandas`, `matplotlib`, and `seaborn` for data exploration and visualization. Here's an example of loading a dataset with `pandas` and creating a simple plot:

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

# Create a sample DataFrame
data = pd.DataFrame({
    "column1": np.random.rand(50),  # 50 random values for column1
    "column2": np.random.rand(50) * 10  # 50 random values (multiplied by 10) for column2
})

# Display the first few rows
print(data.head())

# Create a scatter plot
plt.scatter(data["column1"], data["column2"])
plt.xlabel("Column 1")
plt.ylabel("Column 2")
plt.title("Scatter Plot")
plt.show()
```

This code now generates a sample DataFrame with two columns, `column1` and `column2`, containing random values. The rest of the code remains the same, demonstrating how to display the DataFrame's contents and create a scatter plot using the generated data.

To save your notebook, click the save icon in the toolbar or use the `Ctrl + S` shortcut. Don't forget to rename your Notebook. You can right-click on the Notebook tab or the Notebook in the file browser.
## Restarting the Kernel

`JupyterLab` uses a `kernel` to run your code. The `kernel` is a separate process responsible for executing code and maintaining the state of your computations. Sometimes, you may need to reset your environment if it becomes cluttered with variables or you encounter unexpected behavior.

Restarting the `kernel` clears all variables, functions, and imported modules from memory, allowing you to start fresh without shutting down `JupyterLab` entirely.

To restart the `kernel`:

1. Open the `Kernel` menu in the top toolbar.
2. Select `Restart Kernel` to reset the environment while preserving cell outputs, or `Restart Kernel and Clear All Outputs` to also remove all previously generated outputs from the notebook.

After restarting, re-run any cells containing variable definitions, imports, or computations to restore the environment. This ensures that the notebook state accurately reflects the code you have most recently executed.


---

# Python Libraries for AI

Python is a versatile programming language widely used in Artificial Intelligence (AI) due to its rich library ecosystem that provides efficient and user-friendly tools for developing AI applications. This section focuses on two prominent Python libraries for AI development: `Scikit-learn` and `PyTorch`.

Just a quick note. This section provides a high-level overview of key Python libraries for AI, aiming to familiarize you with their purpose, structure, and common use cases. It offers a foundation for identifying relevant APIs and understanding the general landscape of these libraries. The official documentation will be your best resource to learning every small detail about the libraries. You do not need to copy and run these code snippets.

## Scikit-learn

`Scikit-learn` is a comprehensive library built on `NumPy`, `SciPy`, and `Matplotlib`. It offers a wide range of algorithms and tools for machine learning tasks and provides a consistent and intuitive API, making implementing various machine learning models easy.

- `Supervised Learning:` `Scikit-learn` provides a vast collection of supervised learning algorithms, including:
    - `Linear Regression`
    - `Logistic Regression`
    - `Support Vector Machines (SVMs)`
    - `Decision Trees`
    - `Naive Bayes`
    - `Ensemble Methods` (e.g., Random Forests, Gradient Boosting)
- `Unsupervised Learning:` It also offers various unsupervised learning algorithms, such as:
    - `Clustering` (K-Means, DBSCAN)
    - `Dimensionality Reduction` (PCA, t-SNE)
- `Model Selection and Evaluation:` `Scikit-learn` includes tools for model selection, hyperparameter tuning, and performance evaluation, enabling developers to optimize their models effectively.
- `Data Preprocessing:` It provides functionalities for data preprocessing, including:
    - Feature scaling and normalization
    - Handling missing values
    - Encoding categorical variables

### Data Preprocessing

`Scikit-learn` offers a rich set of tools for preprocessing data, a crucial step in preparing data for machine learning algorithms. These tools help transform raw data into a suitable format that improves the accuracy and efficiency of models.

Feature scaling is essential to ensure that all features have a similar scale, preventing features with larger values from dominating the learning process. `Scikit-learn` provides various scaling techniques:

- `StandardScaler` : Standardizes features by removing the mean and scaling to unit variance.
- `MinMaxScaler` : Scales features to a given range, typically between 0 and 1.
- `RobustScaler` : Scales features using statistics that are robust to outliers.

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
```

Categorical features, representing data in categories or groups, need to be converted into numerical representations for machine learning algorithms to process them. `Scikit-learn` offers encoding techniques:

- `OneHotEncoder` : Creates binary (0 or 1) columns for each category.
- `LabelEncoder` : Assigns a unique integer to each category.

```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder()
X_encoded = encoder.fit_transform(X)
```

Real-world datasets often contain missing values. `Scikit-learn` provides methods to handle these missing values:

- `SimpleImputer` : Replaces missing values with a specified strategy (e.g., mean, median, most frequent).
- `KNNImputer` : Imputes missing values using the k-Nearest Neighbors algorithm.

```python
from sklearn.impute import SimpleImputer

imputer = SimpleImputer(strategy='mean')
X_imputed = imputer.fit_transform(X)
```

### Model Selection and Evaluation

`Scikit-learn` offers tools for selecting the best model and evaluating its performance.

Splitting data into training and testing sets is crucial to evaluating the model's generalization ability to unseen data.

```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
```

Cross-validation provides a more robust evaluation by splitting the data into multiple folds and training/testing on different combinations.

```python
from sklearn.model_selection import cross_val_score

scores = cross_val_score(model, X, y, cv=5)
```

`Scikit-learn` provides various metrics to evaluate model performance:

- `accuracy_score` : For classification tasks.
- `mean_squared_error` : For regression tasks.
- `precision_score`, `recall_score`, `f1_score` : For classification tasks with imbalanced classes.

```python
from sklearn.metrics import accuracy_score

accuracy = accuracy_score(y_test, y_pred)
```

### Model Training and Prediction

`Scikit-learn` follows a consistent API for training and predicting with different models.

Create an instance of the desired model with appropriate hyperparameters.

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(C=1.0)
```

Train the model using the `fit()` method with the training data.

```python
model.fit(X_train, y_train)
```

Make predictions on new data using the `predict()` method.

```python
y_pred = model.predict(X_test)
```

## PyTorch

`PyTorch` is an open-source machine learning library developed by Facebook's AI Research lab. It provides a flexible and powerful framework for building and deploying various types of machine learning models, including deep learning models.

### Key Features

- `Deep Learning:` `PyTorch` excels in deep learning, enabling the development of complex neural networks with multiple layers and architectures.
- `Dynamic Computational Graphs:` Unlike static computational graphs used in libraries like TensorFlow, `PyTorch` uses dynamic computational graphs, which allow for more flexible and intuitive model building and debugging.
- `GPU Support:` `PyTorch` supports GPU acceleration, significantly speeding up the training process for computationally intensive models.
- `TorchVision Integration:` `TorchVision` is a library integrated with `PyTorch` that provides a user-friendly interface for image datasets, pre-trained models, and common image transformations.
- `Automatic Differentiation:` `PyTorch` uses `autograd` to automatically compute gradients, simplifying the process of backpropagation.
- `Community and Ecosystem:` `PyTorch` has a large and active community, leading to a rich ecosystem of tools, libraries, and resources.

### Dynamic Computational Graphs and Tensors

At the heart of `PyTorch` lies the concept of dynamic computational graphs. A dynamic computational graph is created on the fly during the forward pass, allowing for more flexible and dynamic model building. This makes it easier to implement complex and non-linear models.

`Tensors` are multi-dimensional arrays that hold the data being processed. They can be constants, variables, or placeholders. `PyTorch` tensors are similar to NumPy arrays but can run on GPUs for faster computation.

```python
import torch

# Creating a tensor
x = torch.tensor([1.0, 2.0, 3.0])

# Tensors can be moved to GPU if available
if torch.cuda.is_available():
    x = x.to('cuda')
```

### Building Models with PyTorch

`PyTorch` provides a flexible and intuitive interface for building and training deep learning models. The `torch.nn` module contains various layers and modules for constructing neural networks.

The `Sequential` API allows building models layer by layer, adding each layer sequentially.

```python
import torch.nn as nn

model = nn.Sequential(
    nn.Linear(784, 128),
    nn.ReLU(),
    nn.Linear(128, 10),
    nn.Softmax(dim=1)
)
```

The `Module` class provides more flexibility for building complex models with non-linear topologies, shared layers, and multiple inputs/outputs.

```python
import torch.nn as nn

class CustomModel(nn.Module):
    def __init__(self):
        super(CustomModel, self).__init__()
        self.layer1 = nn.Linear(784, 128)
        self.relu = nn.ReLU()
        self.layer2 = nn.Linear(128, 10)
        self.softmax = nn.Softmax(dim=1)

    def forward(self, x):
        x = self.layer1(x)
        x = self.relu(x)
        x = self.layer2(x)
        x = self.softmax(x)
        return x

model = CustomModel()
```

### Training and Evaluation

`PyTorch` provides tools for training and evaluating models.

`Optimizers` are algorithms that adjust the model's parameters during training to minimize the loss function. `PyTorch` offers various optimizers:

- `Adam`
- `SGD` (Stochastic Gradient Descent)
- `RMSprop`

```python
import torch.optim as optim

optimizer = optim.Adam(model.parameters(), lr=0.001)
```

`Loss Functions` measure the difference between the model's predictions and the actual target values. `PyTorch` provides a variety of loss functions:

- `CrossEntropyLoss` : For multi-class classification.
- `BCEWithLogitsLoss` : For binary classification.
- `MSELoss` : For regression.

```python
import torch.nn as nn

loss_fn = nn.CrossEntropyLoss()
```

`Metrics` evaluate the model's performance during training and testing.

- `Accuracy`
- `Precision`
- `Recall`

```python
def accuracy(output, target):
    _, predicted = torch.max(output, 1)
    correct = (predicted == target).sum().item()
    return correct / len(target)
```

The training loop updates the model's parameters based on the training data.

```python
import torch

epochs = 10
num_batches = 100

for epoch in range(epochs):
    for batch in range(num_batches):
        # Get batch of data
        x_batch, y_batch = get_batch(batch)
        
        # Forward pass
        y_pred = model(x_batch)
        
        # Calculate loss
        loss = loss_fn(y_pred, y_batch)
        
        # Backward pass and optimization
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        # Optional: print loss or other metrics
        if batch % 10 == 0:
            print(f'Epoch [{epoch+1}/{epochs}], Batch [{batch+1}/{num_batches}], Loss: {loss.item():.4f}')
```

### Data Loading and Preprocessing

`PyTorch` provides the `torch.utils.data.Dataset` and `DataLoader` classes for handling data loading and preprocessing.

```python
from torch.utils.data import Dataset, DataLoader

class CustomDataset(Dataset):
    def __init__(self, data, labels):
        self.data = data
        self.labels = labels

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx], self.labels[idx]

# Example usage
dataset = CustomDataset(data, labels)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True)
```

### Model Saving and Loading

`PyTorch` allows models to be saved and loaded for inference or further training.

```python
# Save model
torch.save(model.state_dict(), 'model.pth')

# Load model
model = CustomModel()
model.load_state_dict(torch.load('model.pth'))
model.eval()  # Set the model to evaluation mode
```


---

# Datasets


In AI, the quality and characteristics of the data used to train models significantly impact their performance and accuracy. `Datasets`, which are collections of data points used for analysis and model training, come in various forms and formats, each with its own properties and considerations. `Data preprocessing` is a crucial step in the machine-learning pipeline that involves transforming raw data into a suitable format for algorithms to process effectively.

## Understanding Datasets

`Datasets` are structured collections of data used for analysis and model training. They come in various forms, including:

- `Tabular Data`: Data organized into tables with rows and columns, common in spreadsheets or databases.
- `Image Data`: Sets of images represented numerically as pixel arrays.
- `Text Data`: Unstructured data composed of sentences, paragraphs, or full documents.
- `Time Series Data`: Sequential data points collected over time, emphasizing temporal patterns.

The quality of a dataset is fundamental to the success of any data analysis or machine learning project. Here’s why:

- `Model Accuracy`: High-quality datasets produce more accurate models. Poor-quality data—such as noisy, incomplete, or biased datasets—leads to reduced model performance.
- `Generalization`: Carefully curated datasets enable models to generalize effectively to unseen data. This minimizes overfitting and ensures consistent performance in real-world applications.
- `Efficiency`: Clean, well-prepared data reduces both training time and computational demands, streamlining the entire process.
- `Reliability`: Reliable datasets lead to trustworthy insights and decisions. In critical domains like healthcare or finance, data quality directly affects the dependability of results.

### What Makes a Dataset 'Good'

Several key attributes characterize a good dataset:

|Attribute|Description|Example|
|---|---|---|
|`Relevance`|The data should be relevant to the problem at hand. Irrelevant data can introduce noise and reduce model performance.|Text data from social media posts is more relevant than stock market prices for a sentiment analysis task.|
|`Completeness`|The dataset should have minimal missing values. Missing data can lead to biased models and incorrect predictions.|Techniques like imputation can handle missing values, but it's best to start with a complete dataset if possible.|
|`Consistency`|Data should be consistent in format and structure. Inconsistencies can cause errors during preprocessing and model training.|Ensure that date formats are uniform across the dataset (e.g.,`YYYY-MM-DD`).|
|`Quality`|The data should be accurate and free from errors. Errors can arise from data collection, entry, or transmission issues.|Data validation and verification processes can help ensure data quality.|
|`Representativeness`|The dataset should be representative of the population it aims to model. A biased or unrepresentative dataset can lead to biased models.|A facial recognition system's dataset should include a diverse range of faces from different ethnicities, ages, and genders.|
|`Balance`|The dataset should be balanced, especially for classification tasks. Imbalanced datasets can lead to biased models that perform poorly on minority classes.|Techniques like oversampling, undersampling, or generating synthetic data can help balance the dataset.|
|`Size`|The dataset should be large enough to capture the complexity of the problem. Small datasets may not provide enough information for the model to learn effectively.|However, large datasets can also be computationally expensive and require more powerful hardware.|

## The Dataset

The provided dataset, [demo_dataset.csv](https://academy.hackthebox.com/storage/modules/292/demo_dataset.zip) is a `CSV` file containing network log entries. Each record describes a network event and includes details such as the source IP address, destination port, protocol used, the volume of data transferred, and an associated threat level. Analyzing these entries allows one to simulate various network scenarios that are useful for developing and evaluating intrusion detection systems.

### Dataset Structure

The dataset consists of multiple columns, each serving a specific purpose:

- `log_id`: Unique identifier for each log entry.
- `source_ip`: Source IP address for the network event.
- `destination_port`: Destination port number used by the event.
- `protocol`: Network protocol employed (e.g., `TCP`, `TLS`, `SSH`).
- `bytes_transferred`: Total bytes transferred during the event.
- `threat_level`: Indicator of the event's severity. `0` denotes normal traffic, `1` indicates low-threat activity, and `2` signifies a high-threat event.

### Challenges and Considerations

Before processing, it is essential to note potential difficulties:

- The dataset contains a mix of numerical and categorical data.
- Missing values and invalid entries appear in some columns, requiring data cleaning.
- Certain numeric columns may contain non-numeric strings, which must be converted or removed.
- The `threat_level` column includes unknown values (e.g., `?`, `-1`) that must be standardized or addressed during preprocessing.

Acknowledging these challenges early allows the data to be properly cleaned and transformed, facilitating accurate and reliable analysis.

## Loading the Dataset

We first load it into a `pandas DataFrame` to begin working with the dataset. A `pandas DataFrame` is a flexible, two-dimensional labeled data structure that supports a variety of operations for data exploration and preprocessing. Key advantages include labeled axes, heterogeneous data handling, and integration with other Python libraries.

Utilizing a DataFrame simplifies subsequent tasks like inspection, cleaning, encoding, and data transformation.

Code: python

```python
import pandas as pd

# Load the dataset
data = pd.read_csv("./demo_dataset.csv")
```

In this code, `pd.read_csv("./demo_dataset.csv")` loads the downloaded CSV file into a DataFrame named `data`. From here, inspecting, manipulating, and preparing the dataset for further steps in the analysis pipeline becomes straightforward.

## Exploring the Dataset

After loading the dataset, we employ various operations to understand its structure, identify anomalies, and determine the nature of cleaning or transformations needed.

### Viewing Sample Entries

We examine the first few rows to get a quick overview, which can help detect obvious issues like unexpected column names, incorrect data types, or irregular patterns.

Code: python

```python
# Display the first few rows of the dataset
print(data.head())
```

This command outputs the initial rows of the DataFrame, offering an immediate glimpse into the dataset's overall organization.

### Inspecting Data Structure and Types

Understanding the data types and completeness of each column is essential. We can quickly review the dataset's information, including which columns have null values and the total number of entries per column.

Code: python

```python
# Get a summary of column data types and non-null counts
print(data.info())
```

The `info()` method reveals the dataset's shape, column names, data types, and how many entries are present for each column, enabling early detection of columns with missing or unexpected data.

### Checking for Missing Values

Missing values or anomalies must be handled to maintain the dataset's integrity. The next step is to identify how many missing values each column contains.

Code: python

```python
# Identify columns with missing values
print(data.isnull().sum())
```

This command returns the count of null values for each column, helping to prioritize which features need attention. Addressing these missing values may involve imputation, removal, or other cleaning strategies to ensure the dataset remains reliable and valid for further analysis.


---

# Data Preprocessing


`Data preprocessing` transforms raw data into a suitable format for machine learning algorithms. Key techniques include:

- `Data Cleaning`: Handling missing values, removing duplicates, and smoothing noisy data.
- `Data Transformation`: Normalizing, encoding, scaling, and reducing data.
- `Data Integration`: Merging and aggregating data from multiple sources.
- `Data Formatting`: Converting data types and reshaping data structures.

Effective preprocessing addresses inconsistencies, missing values, outliers, noise, and feature scaling, improving the accuracy, efficiency, and robustness of machine learning models.

## Identifying Invalid Values

In addition to missing values, we need to check for invalid values in specific columns. Here are some common checks for the given dataset.

### Checking for Invalid IP Addresses

To identify invalid `source_ip` values, you can use a regular expression to validate the IP addresses:

Code: python

```python
import re

def is_valid_ip(ip):
    pattern = re.compile(r'^((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$')
    return bool(pattern.match(ip))

# Check for invalid IP addresses
invalid_ips = data[~data['source_ip'].astype(str).apply(is_valid_ip)]
print(invalid_ips)
```

### Checking for Invalid Port Numbers

To identify invalid `destination_port` values, you can check if the port numbers are within the valid range (0-65535):

Code: python

```python
def is_valid_port(port):
    try:
        port = int(port)
        return 0 <= port <= 65535
    except ValueError:
        return False

# Check for invalid port numbers
invalid_ports = data[~data['destination_port'].apply(is_valid_port)]
print(invalid_ports)
```

### Checking for Invalid Protocol Values

To identify invalid `protocol` values, you can check against a list of known protocols:

Code: python

```python
valid_protocols = ['TCP', 'TLS', 'SSH', 'POP3', 'DNS', 'HTTPS', 'SMTP', 'FTP', 'UDP', 'HTTP']

# Check for invalid protocol values
invalid_protocols = data[~data['protocol'].isin(valid_protocols)]
print(invalid_protocols)
```

### Checking for Invalid Bytes Transferred

To identify invalid `bytes_transferred` values, you can check if the values are numeric and non-negative:

Code: python

```python
def is_valid_bytes(bytes):
    try:
        bytes = int(bytes)
        return bytes >= 0
    except ValueError:
        return False

# Check for invalid bytes transferred
invalid_bytes = data[~data['bytes_transferred'].apply(is_valid_bytes)]
print(invalid_bytes)
```

### Checking for Invalid Threat Levels

To identify invalid `threat_level` values, you can check if the values are within a valid range (e.g., 0-2):

Code: python

```python
def is_valid_threat_level(threat_level):
    try:
        threat_level = int(threat_level)
        return 0 <= threat_level <= 2
    except ValueError:
        return False

# Check for invalid threat levels
invalid_threat_levels = data[~data['threat_level'].apply(is_valid_threat_level)]
print(invalid_threat_levels)
```

## Handling Invalid Entries

There are a few different ways we can approach this bad data.

### Dropping Invalid Entries

The most straightforward approach is to discard the invalid entries entirely. This ensures that the remaining dataset is clean and free of potentially misleading information.

Code: python

```python
# the ignore errors covers the fact that there might be some overlap between indexes that match other invalid criteria
data = data.drop(invalid_ips.index, errors='ignore') 
data = data.drop(invalid_ports.index, errors='ignore')
data = data.drop(invalid_protocols.index, errors='ignore')
data = data.drop(invalid_bytes.index, errors='ignore')
data = data.drop(invalid_threat_levels.index, errors='ignore')

print(data.describe(include='all'))
```

This method is generally preferred when data accuracy is paramount, and the loss of some data points does not significantly compromise the overall analysis. However, it may not always be feasible, especially if the dataset is small or the invalid entries constitute a substantial portion of the data.

After dropping the bad data from our dataset, we are only left with 77 clean entries.

It is sometimes possible to clean or transform invalid entries into valid and usable data instead of discarding them. This approach aims to retain as much information as possible from the dataset.

### Imputing Missing Values

`Imputing` is the process of replacing missing or invalid values in a dataset with estimated values. This is crucial for maintaining the integrity and usability of the data, especially in machine learning and data analysis tasks where missing values can lead to biased or inaccurate results.

First, convert all invalid or corrupted entries, such as `MISSING_IP`, `INVALID_IP`, `STRING_PORT`, `UNUSED_PORT`, `NON_NUMERIC`, or `?`, into `NaN`. This approach standardizes the representation of missing values, enabling uniform downstream imputation steps.

Code: python

```python
import pandas as pd
import numpy as np
import re
from ipaddress import ip_address

df = pd.read_csv('demo_dataset.csv')

invalid_ips = ['INVALID_IP', 'MISSING_IP']
invalid_ports = ['STRING_PORT', 'UNUSED_PORT']
invalid_bytes = ['NON_NUMERIC', 'NEGATIVE']
invalid_threat = ['?']

df.replace(invalid_ips + invalid_ports + invalid_bytes + invalid_threat, np.nan, inplace=True)

df['destination_port'] = pd.to_numeric(df['destination_port'], errors='coerce')
df['bytes_transferred'] = pd.to_numeric(df['bytes_transferred'], errors='coerce')
df['threat_level'] = pd.to_numeric(df['threat_level'], errors='coerce')

def is_valid_ip(ip):
    pattern = re.compile(r'^((25[0-5]|2[0-4][0-9]|[01]?\d?\d)\.){3}(25[0-5]|2[0-4]\d|[01]?\d?\d)$')
    if pd.isna(ip) or not pattern.match(str(ip)):
        return np.nan
    return ip

df['source_ip'] = df['source_ip'].apply(is_valid_ip)
```

After this step, `NaN` represents all missing or invalid data points.

For basic numeric columns like `bytes_transferred`, use simple methods such as the median or mean. For categorical columns like `protocol`, use the most frequent value.

Code: python

```python
from sklearn.impute import SimpleImputer

numeric_cols = ['destination_port', 'bytes_transferred', 'threat_level']
categorical_cols = ['protocol']

num_imputer = SimpleImputer(strategy='median')
df[numeric_cols] = num_imputer.fit_transform(df[numeric_cols])

cat_imputer = SimpleImputer(strategy='most_frequent')
df[categorical_cols] = cat_imputer.fit_transform(df[categorical_cols])
```

These imputations ensure that all columns have valid, non-missing values, though they do not consider complex relationships among features.

For more sophisticated scenarios, employ advanced techniques like `KNNImputer` or `IterativeImputer`. These methods consider relationships among features to produce contextually meaningful imputations.

Code: python

```python
from sklearn.impute import KNNImputer

knn_imputer = KNNImputer(n_neighbors=5)
df[numeric_cols] = knn_imputer.fit_transform(df[numeric_cols])
```

After cleaning and imputations, apply domain knowledge. For `source_ip` values that remain missing, assign a default such as `0.0.0.0`. Validate `protocol` values against known valid protocols. For ports, ensure values fall within the valid range `0-65535`, and for protocols that imply certain ports, consider mode-based assignments or domain-specific mappings.

Code: python

```python
valid_protocols = ['TCP', 'TLS', 'SSH', 'POP3', 'DNS', 'HTTPS', 'SMTP', 'FTP', 'UDP', 'HTTP']
df.loc[~df['protocol'].isin(valid_protocols), 'protocol'] = df['protocol'].mode()[0]

df['source_ip'] = df['source_ip'].fillna('0.0.0.0')
df['destination_port'] = df['destination_port'].clip(lower=0, upper=65535)
```

Perform final verification steps to confirm that distributions are reasonable and categorical sets remain valid. Adjust imputation strategies and transformations or remove problematic records if anomalies persist.

Code: python

```python
print(df.describe(include='all'))
```


--- 

# Data Transformation

`Data transformations` improve the representation and distribution of features, making them more suitable for machine learning models. These transformations ensure that models can efficiently capture underlying patterns by converting categorical variables into machine-readable formats and addressing skewed numerical distributions. They also enhance trained models' stability, interpretability, and predictive performance.

## Encoding Categorical Features

`Encoding` converts categorical values into numeric form so machine learning algorithms can utilize these features. Depending on the situation, you can choose:

- `OneHotEncoder` for binary indicator features that represent each category separately.
- `LabelEncoder` for integer codes, though this may imply unintended order.
- `HashingEncoder` or frequency-based methods to handle high-cardinality features and control feature space size.

After encoding, verify that the transformed features are meaningful and do not introduce artificial ordering.

### One-Hot Encoding

`One-hot encoding` takes a categorical feature and converts it into a set of new binary features, where each binary feature corresponds to one possible category value. This process creates a set of indicator columns that hold `1` or `0`, indicating the presence or absence of a particular category in each row.

For example, consider the categorical feature `color`, which can take on the values `red`, `green`, or `blue`. In a dataset, you might have rows where `color` is `red` in one instance, `green` in another, and so on. By applying `one-hot encoding`, instead of keeping a single column with values like `red`, `green`, or `blue`, the encoding creates three new binary columns:

- `color_red`
- `color_green`
- `color_blue`

Each of these new columns corresponds to one of the original categories. If a row had `color` set to `red`, the `color_red` column for that row would be `1`, and the other two columns (`color_green` and `color_blue`) would be `0`. Similarly, if `color` was originally `green`, then the `color_green` column would be `1`, while the `color_red` and `color_blue` columns would be `0`.

![Table showing one-hot encoding of colors: red, green, blue.](https://academy.hackthebox.com/storage/modules/292/data_encoding.png)

This approach prevents models from misinterpreting category values as numeric hierarchies. However, it can increase the number of features if a category has many unique values.

In this case, we are going to encode the `protocol` feature.

Code: python

```python
from sklearn.preprocessing import OneHotEncoder

encoder = OneHotEncoder(handle_unknown='ignore', sparse_output=False)
encoded = encoder.fit_transform(df[['protocol']])

encoded_df = pd.DataFrame(encoded, columns=encoder.get_feature_names_out(['protocol']))
df = pd.concat([df.drop('protocol', axis=1), encoded_df], axis=1)
```

The original `protocol` feature is replaced with distinct binary columns, ensuring the model interprets each category independently.

## Handling Skewed Data

When a feature is `skewed`, its values are unevenly distributed, often with most observations clustered near one end and a few extreme values stretching out the distribution. Such skew can affect the performance of machine learning models, especially those sensitive to outliers or that assume more uniform or normal-like data distributions.

`Scaling` or transforming these skewed features helps models better capture patterns in the data. One common transformation is applying a `log` transform to compress large values more than small ones, resulting in a more balanced distribution and less dominated by outliers. By doing this, models often gain improved stability, accuracy, and generalization ability.

Below, we show how to apply a `log` transform using the `log1p` function. This approach adds 1 to each value before taking the `log`, ensuring that the transform is defined even for values at or near zero.

Code: python

```python
import numpy as np

# Apply logarithmic transformation to a skewed feature to reduce its skewness
df["bytes_transferred"] = np.log1p(df["bytes_transferred"])  # Add 1 to avoid log(0)
```

The code above transforms the `bytes_transferred` feature. Before this transformation, the feature might have had a few very large values, overshadowing the majority of smaller observations. After the transformation, the distribution is evener, helping the model treat all data points fairly and reducing the risk of overfitting outliers.

![Two histograms: original distribution of bytes transferred and log-transformed distribution.](https://academy.hackthebox.com/storage/modules/292/log_histogram.png)

Visual comparisons of the distribution before and after the transform (as shown by the above figure) confirm that the original skew has been substantially reduced. Although no information is lost, the model now views the data through a lens that downplays extreme cases and highlights underlying patterns more clearly.

## Data Splitting

`Data splitting` involves dividing a dataset into three distinct subsets—`training`, `validation`, and `testing`—to ensure reliable model evaluation. By having separate sets, you can train your model on one subset, fine-tune it on another, and finally test its performance on data it has never seen before.

- `Training Set`: Used to fit the model. Typically accounts for around 60-80% of the entire dataset.
- `Validation Set`: Used for tuning hyperparameters and model selection. Often around 10-20% of the entire dataset.
- `Test Set`: Used only after all model selections and tuning are complete. Often around 10-20% of the entire dataset.

The code below demonstrates one approach using `train_test_split` from `scikit-learn`. The initial split allocates 80% of the data for training and 20% for testing. A subsequent split divides the 80% training portion into 60% for final training and 20% for validation.

Note that `test_size=0.25` in the second split refers to 25% of the previously created training subset (which is 80% of the data). In other words, `0.8 × 0.25 = 0.2` (20% of the entire dataset), leaving 60% for training and 20% for validation overall.

Code: python

```python
from sklearn.model_selection import train_test_split

# Separate features (X) and target (y)
X = df.drop("threat_level", axis=1)
y = df["threat_level"]

# Initial split: 80% training, 20% testing
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=1337)

# Second split: from the 80% training portion, allocate 60% for final training and 20% for validation
X_train, X_val, y_train, y_val = train_test_split(X_train, y_train, test_size=0.25, random_state=1337)
```

These subsets support a structured workflow:

- Train the model on `X_train` and `y_train`.
- Tune hyperparameters or compare different models using `X_val` and `y_val`.
- Finally, evaluate the performance on the untouched `X_test` and `y_test`.


---

# Metrics for Evaluating a Model


When assessing a trained machine learning model, one examines a set of numerical metrics to gauge how well the model performs on a given task. These metrics often quantify the relationship between predictions and known ground-truth labels.

In the [Fundamentals of AI](https://academy.hackthebox.com/module/details/290) module, we briefly covered metrics such as `accuracy`, `precision`, `recall`, and `F1-score`, and we know that these metrics provide different perspectives on model behavior.

## Accuracy

`Accuracy` is the proportion of correct predictions out of all predictions made. It measures how often the model classifies instances correctly. A model with `accuracy: 0.9950` indicates that it makes correct predictions 99.50% of the time.

Key points about `accuracy`:

- Measures overall correctness.
- Computed as `(true positives + true negatives) / (all instances)`.
- May be misleading in cases of class imbalance.

While `accuracy` appears intuitive, relying on it alone can hide important details. Consider a spam classification scenario where only 1% of incoming emails are spam and 99% are legitimate. A model that always predicts every email as legitimate will achieve `accuracy: 0.99`, but it will never catch any spam.

In this case, accuracy fails to highlight the model’s inability to correctly identify the minority class. This underscores the importance of complementary metrics, such as `precision`, `recall`, or `F1-score`, which provide a more nuanced understanding of performance when dealing with imbalanced datasets.

## Precision

`Precision` measures how often the model’s predicted positives are truly positive. For `precision: 0.9949`, when the model labels an instance as positive, it is correct 99.49% of the time.

Key points about `precision`:

- Reflects quality of positive predictions.
- Computed as `true positives / (true positives + false positives)`.
- High `precision` reduces wasted effort caused by false alarms.

With the spam classification example, if the model labels 100 emails as spam, and 99 of them are actually spam, then its `precision` is high. This reduces the inconvenience of losing important, legitimate emails to the spam folder. However, if the model rarely identifies spam in the first place, it may fail to catch a large portion of malicious emails. High `precision` alone does not guarantee that the model is finding all the spam it should.

## Recall

`Recall` measures the model’s ability to identify all positive instances. For `recall: 0.9950`, the model detects 99.50% of all positives.

Key points about `recall`:

- Reflects completeness of positive detection.
- Computed as `true positives / (true positives + false negatives)`.
- High `recall` reduces the risk of missing critical cases.

In the spam classification scenario, a model with high `recall` correctly flags most spam emails. This helps ensure that suspicious content does not slip through unnoticed. However, a model with very high `recall` but low `precision` might flood the spam folder with benign emails. Although it rarely misses spam, it inconveniences the user by misclassifying too many legitimate emails as spam.

## F1-Score

`F1-score` is the harmonic mean of `precision` and `recall`. For `F1-score: 0.9949`, the metric indicates a near-perfect balance between these two aspects.

Key points about `F1-score`:

- Balances `precision` and `recall`.
- Computed as `2 * (precision * recall) / (precision + recall)`.
- Useful for tasks involving class imbalance.

Continuing with the spam classification scenario, the `F1-score` ensures that the model not only minimizes the misclassification of legitimate emails (high `precision`) but also effectively identifies the majority of spam messages (high `recall`). By focusing on the balance rather than just one metric, the `F1-score` provides a more complete picture of the model’s performance in identifying and correctly handling both spam and non-spam emails.

## Additional Considerations

While these four metrics are common, other measures may provide further insights:

- `Specificity`: Measures how effectively the model identifies negatives.
- `AUC`: The Area Under the ROC Curve, indicating the model’s discriminative capability at various thresholds.
- `Matthews Correlation Coefficient`: Useful for highly imbalanced datasets.
- `Confusion Matrix`: Summarizes predictions versus true labels, offering a comprehensive view of performance.

Such metrics and visualizations help confirm that the given high values truly reflect robust performance, not just favorable conditions in the dataset.

## Contextualizing the Metrics

When evaluating a model’s metrics (`accuracy: 0.9750`, `precision: 0.9300`, `recall: 0.9100`, `F1-score: 0.9200`), consider the following:

- Are these metrics consistent across different segments of the data?
- Does the dataset represent real-world conditions, including the presence of class imbalances?
- Are external factors, such as the cost of false positives or false negatives, properly accounted for?

Even metrics that look impressive may not fully capture real-world performance if the dataset does not reflect operational conditions. For instance, high `accuracy` could be achieved if negative cases are heavily overrepresented, making it easier to appear correct by default. Verifying that both `precision` and `recall` remain robust helps ensure the model identifies important instances without becoming overwhelmed by incorrect predictions.

Depending on the setting, certain trade-offs emerge:

- In threat detection, a model might favor `recall` to avoid missing critical threats, even if it occasionally flags benign events.
- In environments with limited resources, focusing on `precision` can reduce the burden caused by following up on false alarms.

These metrics, considered together, provide a balanced perspective. The relatively high and reasonably aligned `precision` and `recall` values yield a strong `F1-score`, suggesting that the model performs consistently well across different classes. This balanced performance supports confidence that the model’s decisions are both reliable and meaningful in practice.