# -*- coding: utf-8 -*-
"""DENORMALIZED_StocksRVFL+.ipynb

Automatically generated by Colaboratory.

Original file is located at
    [https://colab.research.google.com/drive/15epCvhHmwQmN4JM8RcloANE8142sxhJ1](https://colab.research.google.com/drive/15epCvhHmwQmN4JM8RcloANE8142sxhJ1)

# Extracting Historical data FROM URL AND SAVING IT INTO 2D-MATRIX

```python
import yfinance as yf
from datetime import datetime

def get_daily_stock_data(symbol, start_date, end_date):
    try:
        stock_data = yf.download(symbol, start=start_date, end=end_date)
        return stock_data
    except Exception as e:
        print(f"An error occurred: {e}")
        return None

# Example usage:
symbol = 'TATAMOTORS.NS'

# Set the start and end dates
start_date = '2018-11-10'
end_date = '2023-12-10'

daily_data = get_daily_stock_data(symbol, start_date, end_date)

if daily_data is not None and not daily_data.empty:
    print(f"Daily Stock Data for {symbol} (from {start_date} to {end_date}):")
    print(daily_data.head())
else:
    print(f"Failed to retrieve daily stock data for {symbol}.")

Importing libraries
python
Copy code
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

data = daily_data
X_input Data preparation
python
Copy code
# Function to create sliding windows for the 'Close' column
def create_sliding_windows(data_column, window_size):
    windows = []
    for i in range(len(data_column) - window_size + 1):
        window = data_column.iloc[i:i + window_size]
        windows.append(window.values.flatten())
    return pd.DataFrame(windows)

# Set your desired window size
window_size = 5

# Create sliding windows for the 'Close' column
X_input = create_sliding_windows(data['Close'], window_size)

# Display the resulting 2D matrix
print(X_input[:5])
Row-wise normalization
python
Copy code
import pandas as pd
import numpy as np

# Assuming 'data' is a 1231x5 DataFrame
# Set your desired window size
window_size = 5

# Function to normalize each row independently to the range [0, 1]
def row_wise_normalization(X_input):
    normalized_windows = []

    for i in range(len(X_input)):
        # Extract features (X) and target variable (Y)
        X_window = X_input.iloc[i, :-1]
        Y_window = X_input.iloc[i, -1]

        # Store X_min and X_max
        X_min = X_window.min()
        X_max = X_window.max()
        X_min_list.append(X_min)
        X_max_list.append(X_max)

        # Normalize each element in the row of X to the range [0, 1]
        X_normalized = (X_window - X_min) / (X_max - X_min)

        # Normalize the last element in the row of Y to the range [0, 1]
        Y_normalized = (Y_window - X_min) / (X_max - X_min)

        # Append the normalized X and Y to the list
        normalized_row = np.concatenate([X_normalized.values, np.array([Y_normalized])])
        normalized_windows.append(normalized_row)

    # Create arrays from the lists
    X_min_array = np.array(X_min_list)
    X_max_array = np.array(X_max_list)

    # Create a matrix from the list of normalized rows
    normalized_data = np.array(normalized_windows)

    return normalized_data, X_min_array, X_max_array

# Apply row-wise normalization
normalized_windows, X_min_array, X_max_array = row_wise_normalization(X_input)

# Display the resulting 2D matrix with normalized values
print("Normalized Data:")
print(normalized_windows[:5])
print(normalized_windows.shape)
X_previleged data preparation
python
Copy code
# Function to create sliding windows for the 'Volume' column
def create_sliding_windows(data_column, window_size):
    windows = []
    for i in range(len(data_column) - window_size + 1):
        window = data_column.iloc[i:i + window_size]
        windows.append(window.values.flatten())
    return pd.DataFrame(windows)

# Set your desired window size
window_size = 5

# Create sliding windows for the 'Close' column
previledged = create_sliding_windows(data['Volume'], window_size)

# Display the resulting 2D matrix
print(previledged[:5])
Row-wise normalization
python
Copy code
import pandas as pd
import numpy as np

# Assuming 'data' is a 1231x5 DataFrame
# Set your desired window size
window_size = 5

# Function to normalize each row independently to the range [0, 1]
def row_wise_normalization(previledged):
    normalized_windows = []

    for i in range(len(previledged)):
        # Extract features (X) and target variable (Y)
        X_window = previledged.iloc[i, :-1]
        Y_window = previledged.iloc[i, -1]

        # Normalize each element in the row of X to the range [0, 1]
        X_normalized = (X_window - X_window.min()) / (X_window.max() - X_window.min())

        # Normalize the last element in the row of Y to the range [0, 1]
        Y_normalized = (Y_window - X_window.min()) / (X_window.max() - X_window.min())

        # Append the normalized X and Y to the list
        normalized_row = np.concatenate([X_normalized.values, np.array([Y_normalized])])
        normalized_windows.append(normalized_row)

    # Create a matrix from the list of normalized rows
    normalized_data = np.array(normalized_windows)

    return normalized_data

# Apply row-wise normalization
X_new = row_wise_normalization(previledged)

# Display the resulting 2D matrix with normalized values
print("Normalized Data:")
print(X_new[:5])
print(X_new.shape)
python
Copy code
from sklearn.model_selection import train_test_split

# Splitting the data into test and train

X_new = X_new[:,:-1]

# Split the dataset into training and testing sets
X_privileged, X_privileged_test = train_test_split(X_new,test_size=0.2, random_state=42)

X = normalized_windows[:, :-1]
y = normalized_windows[:, -1]

# Split the dataset into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

print(X_train.shape, X_test.shape, y_train.shape, y_test.shape)

# Split X_min and X_max into training and testing sets
X_min_train, X_min_test, X_max_train, X_max_test = train_test_split(X_min_array, X_max_array, test_size=0.2, random_state=42)
Parameter Tuning
python
Copy code
import numpy as np
from sklearn.model_selection import KFold
from sklearn.metrics import mean_squared_error

def sigmoid(weighted_sum_fold):
    return 1 / (1 + np.exp(-weighted_sum_fold))

def perform_cross_validation(X, Y, hidden_layer_columns_range, A_values):
    num_folds = 10
    kf = KFold(n_splits=num_folds, shuffle=True, random_state=42)

    best_mse = float('inf')
    best_hidden_columns = None
    best_A = None

    for hidden_columns in hidden_layer_columns_range:
        total_mse = 0.0

        for A in A_values:
            for train_index, valid_index in kf.split(X):
                X_train_fold, X_valid_fold = X[train_index], X[valid_index]
                Y_train_fold, Y_valid_fold = Y[train_index], Y[valid_index]

                # Generate random weights for the hidden layer
                hidden_layer_weights_fold = np.random.rand(X_train_fold.shape[1], hidden_columns)

                # Calculate the total weighted sum for the validation set
                hidden_valid_weights_fold = np.dot(X_valid_fold, hidden_layer_weights_fold)
                activation_result_valid_fold = sigmoid(hidden_valid_weights_fold)
                augmented_X_valid_fold = np.hstack((X_valid_fold, activation_result_valid_fold))

                # Calculate output weights for the training set
                D_fold = augmented_X_valid_fold
                D_transpose_fold = np.transpose(D_fold)
                term_inside_parentheses_fold = np.dot(D_fold, D_transpose_fold) + A * np.identity(X_valid_fold.shape[0])
                inverse_term_fold = np.linalg.inv(term_inside_parentheses_fold)
                Half_fold = np.dot(D_transpose_fold, inverse_term_fold)
                output_weights_fold = np.dot(Half_fold, Y_valid_fold)

                # Predict on the validation set
                predicted_output_valid_fold = np.dot(augmented_X_valid_fold, output_weights_fold)

                # Calculate Mean Squared Error for the validation set
                mse_valid = mean_squared_error(Y_valid_fold, predicted_output_valid_fold)
                total_mse += mse_valid

        # Average MSE over the folds and A values
        average_mse = total_mse / (num_folds * len(A_values))

        # Print and update best values if needed
        print(f"For Hidden Columns={hidden_columns}, Average MSE: {average_mse}")

        if average_mse < best_mse:
            best_mse = average_mse
            best_hidden_columns = hidden_columns
            best_A = A

    print(f"Best Average MSE: {best_mse} achieved with Hidden Columns={best_hidden_columns} and Best A={best_A}")
    return best_hidden_columns, best_A

# Assuming X_train and Y_train are already defined
# Define the range of hidden layer columns and A values to test
hidden_columns_range = range(50, 1001, 50)
A_values = np.logspace(-5, 5, 11)

# Apply cross-validation
best_hidden_columns, best_A = perform_cross_validation(X_train, y_train, hidden_columns_range, A_values)
Weight generation X_train
python
Copy code
def sigmoid(activation_result):
    return 1 / (1 + np.exp(-activation_result))

hidden_layer_weights = np.random.rand(X_train.shape[1], best_hidden_columns)
weighted_sum = np.dot(X_train, hidden_layer_weights)
random_biases = np.random.rand(weighted_sum.shape[0], 1)
total_Weight_sum = weighted_sum + random_biases
activation_result = sigmoid(total_Weight_sum)

# Augmenting the input features
H = np.hstack((X_train, activation_result))

Weights generation for X privilege
python
Copy code
hidden_layer_weights_preveledged = np.random.rand(X_privileged.shape[1], best_hidden_columns)
weighted_sum_preveledged = np.dot(X_privileged, hidden_layer_weights_preveledged)

random_biases_prev = np.random.rand(weighted_sum_preveledged.shape[0], 1)
total_Weight_sum_preveledged = weighted_sum_preveledged + random_biases_prev

# Applying sigmoid to the total weighted sum
activation_result_prev = sigmoid(total_Weight_sum_preveledged)

H_prev = np.hstack((X_privileged, activation_result_prev))
RVFL+ BASE FORMULA
python
Copy code
## Assuming A, C, and num_classes are defined
A = best_A  # Specify your coefficient A
C = 1.0  # Specify your coefficient C
L = np.dot(H, H.T) + (1 / C) * np.dot(H_prev, H_prev.T) + (1 / A) * np.identity(H.shape[0])
L_inverse = np.linalg.inv(L)

k = np.dot(H.T, L_inverse)
E = np.dot(H_prev, H_prev.T)
ee = np.ones(E.shape[1])
Ap = np.dot(np.dot(H_prev, H_prev.T), ee)
# Calculate privileged information P
P = y_train - (A / C) * Ap
w = np.dot(k, P)

w[:5]
Testing data formulation previlege data
python
Copy code
# ... (previous code)

hidden_layer_weight_test_p = np.random.rand(X_privileged_test.shape[1], best_hidden_columns)
# Now, 'w' contains the calculated output weights
hidden_test_weights_p = np.dot(X_privileged_test, hidden_layer_weight_test_p)
random_biases_test_p = np.random.rand(hidden_test_weights_p.shape[0], 1)
weighted_sum_test_p = hidden_test_weights_p + random_biases_test_p
activation_result_test_p = sigmoid(weighted_sum_test_p)
augmented_X_test_p = np.hstack((X_privileged_test, activation_result_test_p))

augmented_X_test_p[:5]

# Calculate predicted output
predicted_output_p = np.dot(augmented_X_test_p, w)

predicted_output_p.shape
Normal RVFL output weights calculation
python
Copy code
hidden_layer_weights = np.random.rand(X_train.shape[1], best_hidden_columns)
random_biases = np.random.rand(hidden_layer_weights.shape[0], 1)
hidden_layer_weights = hidden_layer_weights + random_biases

# Calculating the total weighted sum
weighted_sum = np.dot(X_train, hidden_layer_weights)

# Applying sigmoid to the total weighted sum
activation_result = sigmoid(weighted_sum)
# Augmenting the input features
augmented_X_train = np.hstack((X_train, activation_result))

D = augmented_X_train
D_transpose = np.transpose(D)
A = best_A  # Regularization parameter
term_inside_parentheses = np.dot(D, D_transpose) + A * np.identity(X_train.shape[0])
inverse_term = np.linalg.inv(term_inside_parentheses)
Half = np.dot(D_transpose, inverse_term)

# Calculate the output weights using the formula
output_weights = np.dot(Half, y_train)
print("Output weights")
print(output_weights[:5])
print(output_weights.shape)
Testing data formulation Normal RVFL
python
Copy code
# ... (previous code)

hidden_layer_weight_test = np.random.rand(X_test.shape[1], best_hidden_columns)
hidden_test_weights = np.dot(X_test, hidden_layer_weight_test)
random_biases_test = np.random.rand(hidden_test_weights.shape[0], 1)
weighted_sum_test = hidden_test_weights + random_biases_test
activation_result_test = sigmoid(weighted_sum_test)
augmented_X_test = np.hstack((X_test, activation_result_test))

predicted_output_test = np.dot(augmented_X_test, output_weights)
Augmenting the weights of normal RVFL and X privilege test results
python
Copy code
predicted_output = sigmoid(predicted_output_p) + sigmoid(predicted_output_test)

predicted_output[:5]

y_test[:5]
MSE and Graph representation
python
Copy code
from sklearn.metrics import mean_squared_error
import seaborn as sns
import matplotlib.pyplot as plt

# Calculate Mean Squared Error
mse = mean_squared_error(y_test, predicted_output)
print(f"Mean Squared Error: {mse}")

# Plotting the results
plt.figure(figsize=(10, 6))
sns.lineplot(x=range(len(y_test)), y=y_test, label="Actual Values")
sns.lineplot(x=range(len(predicted_output)), y=predicted_output, label="Predicted Values")

# Set labels and legend
plt.xlabel('Sample Index')
plt.ylabel('Target Value')
plt.legend()
plt.title('Actual vs Predicted Values with MSE')

# Show the plot
plt.show()

X_min_test.shape

# Assuming predicted_output is the array containing predicted values
denormalized_output = []

for i in range(len(X_min_test)):
    # Extract the normalized predicted output
    normalized_prediction = predicted_output[i]

    # Extract the corresponding X_min and X_max values
    X_min = X_min_test[i]
    X_max = X_max_test[i]

    # Denormalize the predicted output
    denormalized_prediction = normalized_prediction * (X_max - X_min) + X_min

    # Append the denormalized prediction to the list
    denormalized_output.append(denormalized_prediction)

# Display the denormalized output
print("Denormalized Output:")
print(denormalized_output[:5])
