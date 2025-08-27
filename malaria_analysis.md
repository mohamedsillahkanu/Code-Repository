**This is a section that talks about how to combine multiple files of the same variables or columns into one**
## Method

**This is one method how to combine files**   


![NMCP lOGO](Concatenate.png)



## Code 

```python
import pandas as pd

# ===== Create dummy data =====
data = {
    "hf_uid": [1]*12 + [2]*12 + [3]*12,
    "date": pd.date_range("2024-01-01", "2024-12-01", freq='MS').tolist()*3,
    "report": [
        # HF 1:
        0,0,0,0,0,0,0,0,1,1,1,1,
        # HF 2:
        1,0,1,0,1,0,0,0,0,0,0,0,
        # HF 3:
        1,1,0,0,0,0,0,0,0,1,1,1
    ]
}

df = pd.DataFrame(data)

# ===== Concept=====


def mark_inactive(group):
    """
    The function works on one health facility at a time. It takes all the data for a single hf_uid and looks at it separately from the others.
    It first sorts the records by month. This ensures that the months are in the correct order, so that when we look for consecutive zeros, we are moving through time correctly.
    It checks which months have report = 0. It makes a list of True and False values, where True means the facility did not report that month (a zero), and False means it did.
    It groups consecutive months with the same status. Each time the pattern changes (from 0 to 1 or 1 to 0), it starts a new group. So, if there are six zeros in a row, they all belong to the same group.
    It counts how many zeros are in each group. For every group of zeros, it calculates the total number of months in that run and attaches that number to each month in that group.
    It decides whether each month is Active or Inactive. If a month is part of a zero group that lasts six months or more, it’s marked as “Inactive.” If it’s part of a shorter run of zeros, or it’s a month with a report (1), it’s marked as “Active.”
    It applies this process to every health facility. The function repeats these steps for each hf_uid and combines the results back into one table. The end result is a dataset where every month is labeled as Active or Inactive, and all months in a long zero streak (six or more) are correctly marked as Inactive.
    """

    # Sort the data by date to ensure the months are in chronological order
    group = group.sort_values('date')

    # Create a boolean series: True where report == 0, False otherwise
    # This helps identify months with zero reports
    zeros = group['report'] == 0

    # Identify consecutive runs of the same value (True or False)
    # zeros != zeros.shift() returns True whenever the current value differs from the previous one
    # cumsum() gives a unique number to each consecutive run of the same value
    count = (zeros != zeros.shift()).cumsum()

    # Calculate the length of each consecutive run of zeros
    # transform('sum') computes the total number of True values in each group
    # This repeats the run length for every month in that run
    run_lengths = zeros.groupby(count).transform('sum')

    # Create a new column 'status' based on the rules:
    # - If the month is a zero (zeros == True) AND the run length >= 6 → 'Inactive'
    # - Otherwise → 'Active'
    # zip(zeros, run_lengths) pairs each month with its run length
    group['status'] = ['Inactive' if z and l >= 6 else 'Active'
                       for z, l in zip(zeros, run_lengths)]

    # Return the modified group with the new 'status' column
    return group

# Apply the function to each health facility separately
# groupby('hf_uid') ensures runs are calculated within each HF only
# reset_index(drop=True) cleans up the index after grouping
df = df.groupby('hf_uid').apply(mark_inactive).reset_index(drop=True)

df_inactive=df[df['status']=='Inactive']
# Display the final DataFrame with status for every month
df_inactive.head(30)
```

