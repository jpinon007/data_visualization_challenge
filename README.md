# data_visualization_challenge
# Background : I have just joined Pymaceuticals, Inc., a new pharmaceutical company that specializes in anti-cancer medications. Recently, it began screening for potential treatments for squamous cell carcinoma (SCC). As a senior data analyst, I must generate tables and figures to report the clinical study of mice who were identified with SCC tumors and received treatment with various drug regimens.
# Note: To help me with this challenge, I used google search, Xpert Learning Assistant, and referenced the solution posted by my instructor on GitLab https://git.bootcampcontent.com/Northwestern-University/NU-VIRT-DATA-PT-06-2024-U-LOLC.git
# Solution: I imported necessary dependencies to help run my code. 
import csv
import matplotlib.pyplot as plt
import pandas as pd
import scipy.stats as st
# Next, I created paths to the data files needed. I then merged the data into a single DataFrame. 
mouse_metadata_path =  r"C:\Users\jasmi\OneDrive\Documents\module_5_hw\data\Mouse_metadata.csv"
study_results_path = r"C:\Users\jasmi\OneDrive\Documents\module_5_hw\data\Study_results.csv"

# Read the mouse data and the study results
mouse_metadata = pd.read_csv(mouse_metadata_path)
study_results = pd.read_csv(study_results_path)

# Combine the data into a single DataFrame
study_data_merge = pd.merge(mouse_metadata, study_results, on="Mouse ID")

# Display the data table for preview
study_data_merge.head()

# I prepared the data by displaying the number of unique mice IDs and checking mice IDs with duplicate time points. Next, I displayed the data with the duplicate mouse ID. I created a clean DataFrame where the duplicate mouse ID data was omitted. I then displayed the number of mice in the clean DataFrame.

# -Summary Stats: I made a table composed of mean. median, variance, standard deviation, and SEm of the tumor volume for each drug regimen. I calculated the statistical properties of each regimen and proceeded to organize the results into a single summary DataFrame. 

drug_reg_mean = clean_study_df.groupby('Drug Regimen')['Tumor Volume (mm3)'].mean()
drug_reg_median = clean_study_df.groupby('Drug Regimen')['Tumor Volume (mm3)'].median()
drug_reg_variance = clean_study_df.groupby('Drug Regimen')['Tumor Volume (mm3)'].var()
drug_reg_sds = clean_study_df.groupby('Drug Regimen')['Tumor Volume (mm3)'].std()
drug_reg_sems = clean_study_df.groupby('Drug Regimen')['Tumor Volume (mm3)'].sem()
drug_reg_summary_table = pd.DataFrame({"Mean Tumor Volume":drug_reg_mean,
                              "Median Tumor Volume":drug_reg_mean,
                              "Tumor Volume Variance":drug_reg_variance,
                              "Tumor Volume Std. Dev.":drug_reg_sds,
                              "Tumor Volume Std. Err.":drug_reg_sems})
drug_reg_summary_table

# Next, I used the aggregation method to produce the same summary stats in a single line. 
stats_summary_table = clean_study_df.groupby("Drug Regimen").agg({"Tumor Volume (mm3)":["mean","median","var","std","sem"]})
stats_summary_table
# Bar Charts: I generated a bar chart to show the total rows in Mouse ID and Timepoints for each drug regimen using Pandas. 
my_colors = [(x/10.0, x/20.0, 0.75) for x in range(len(clean_study_df))]
counts = clean_study_df['Drug Regimen'].value_counts()
counts.plot(kind="bar", x= 'Drug Regimen', y='Timepoints',color= my_colors)
plt.xlabel("Drug Regimen")
plt.xticks(rotation=90)
plt.ylabel("# of Observed Mouse Timepoints")
plt.title("Total Rows Mouse ID & Timepoints")
plt.show()
# I then generated a bar plot visualizing the same data and using Pyplot. 
colors = [(x/10.0, x/20.0, 0.75) for x in range(len(clean_study_df))]

counts = clean_study_df['Drug Regimen'].value_counts()
plt.bar(counts.index.values,counts.values, color=colors)
plt.xlabel("Drug Regimen")
plt.xticks(rotation=90)
plt.ylabel("# of Observed Mouse Timepoints")
plt.title("Total Rows Mouse ID & Timepoints")
plt.show()
# Pie Chart: Using Pandas, I created a pie chart to show the distribution of unique female and male mice in thw study conducted. 
# Get the unique mice with their gender
unique_mice_df = clean_study_df.loc[:, ["Mouse ID", "Sex"]].drop_duplicates()

# Make the pie chart
colors = ['burlywood', 'sienna']
pie_title_font =  {'family': 'serif', 'color':'black', 'weight':'light', 'size': 16}
counts = unique_mice_df.Sex.value_counts()
explode = (0, 0.1) 
counts.plot(kind="pie", autopct='%1.1f%%', shadow=True, colors=colors, explode=explode)
plt.title('Unique Distribution Male vs. Female Mice', fontdict= pie_title_font)
plt.show()
# Next, I used Pyplot to generate a pie chart displaying the same data. 

# Get the unique mice with their gender
mice_df = clean_study_df.loc[:, ["Mouse ID", "Sex"]].drop_duplicates()

# Make the pie chart
counts = mice_df.Sex.value_counts()
labels= ['Male','Female']
explode = (0, 0.1)  
colors = ['xkcd:sky blue','cyan']
pie_title_font =  {'family': 'serif', 'color':'black', 'weight':'light', 'size': 16}
plt.pie(counts.values, labels=labels,labeldistance=.3, autopct='%1.1f%%', colors = colors, explode = explode, shadow = True)
plt.ylabel("Mice Count")
plt.title('Unique Distribution Male vs. Female Mice', fontdict= pie_title_font)
plt.show()
# Quartiles, Outliers, and Boxplots: I worked out the final tumor volume of each mouse from four different drug regimens(Capomulin, Ramicane, Infubinol, and Ceftamin). Then I merged this group DataFrame with the original DataFrame to get the tumor volume at the last timepoint. 
max_tumor_volume = clean_study_df.groupby(["Mouse ID"])['Timepoint'].max()
max_tumor_volume = max_tumor_volume.reset_index()
merged_tumor_df = max_tumor_volume.merge(clean_study_df,on=['Mouse ID','Timepoint'],how="left")
# I calculated the quartiles, lower, upper, IQR, boundaries, and outliers.
# Put treatments into a list for for loop (and later for plot labels)
treatment_list = ["Capomulin", "Ramicane", "Infubinol", "Ceftamin"]

# Create empty list to fill with tumor vol data (for plotting)
tumor_vol_list = []

# Calculate the IQR and quantitatively determine if there are any potential outliers.
for drug in treatment_list:

    # Locate the rows which contain mice on each drug and get the tumor volumes
    final_tumor_vol = merged_tumor_df.loc[merged_tumor_df["Drug Regimen"] == drug, 'Tumor Volume (mm3)']

    # add subset
    tumor_vol_list.append(final_tumor_vol)

    # Determine outliers using upper and lower bounds
    quartiles = final_tumor_vol.quantile([.25,.5,.75])
    lowerq = quartiles[0.25]
    upperq = quartiles[0.75]
    iqr = upperq-lowerq
    lower_bound = lowerq - (1.5*iqr)
    upper_bound = upperq + (1.5*iqr)
    outliers = final_tumor_vol.loc[(final_tumor_vol < lower_bound) | (final_tumor_vol > upper_bound)]
    print(f"{drug}'s potential outliers: {outliers}")
  # Next, I created a boxplot to display the tumor volume distribution per treatment group. 
orange_out = dict(markerfacecolor='magenta',markersize=12)
plt.boxplot(tumor_vol_list, labels = treatment_list,flierprops=orange_out)
plt.ylabel('Final Tumor Volume (mm3)')
plt.title('Tumor Volume Distribution Per Treatment Group')
plt.show()
# Line and Scatter Plots: I made a line plot comparing tumor volumes and time points for a single mouse that was involved in the Capomulin drug regimen. 
capomulin_table = clean_study_df.loc[clean_study_df['Drug Regimen'] == "Capomulin"]
mousedata = capomulin_table.loc[capomulin_table['Mouse ID']== 'l509']
plt.plot(mousedata['Timepoint'],mousedata['Tumor Volume (mm3)'], linewidth=2, color='mediumblue')
plt.xlabel('Timepoint (days)')
plt.ylabel('Tumor Volume (mm3)')
plt.title('Capomulin treatment of mouse l509')
plt.show()
# I created a scatter plot that compared weights and Tumor Volumes for the duration of the Capomulin regimen. 
capomulin_table = clean_study_df.loc[clean_study_df['Drug Regimen'] == "Capomulin"]
capomulin_average = capomulin_table.groupby(['Mouse ID'])[['Weight (g)', 'Tumor Volume (mm3)']].mean()
plt.scatter(capomulin_average['Weight (g)'],capomulin_average['Tumor Volume (mm3)'])
plt.xlabel('Weight (g)')
plt.ylabel('Average Tumor Volume (mm3)')
plt.title("Capomulin Regimen Weight & Tumor Volume Comparison")
plt.show()
# Correlation and Regression: Finally I evaluated the correlation factor and linear regression display for mouse weight and average observed tumor volume during the Capomulin drug regimen. 
corr=round(st.pearsonr(capomulin_average['Weight (g)'],capomulin_average['Tumor Volume (mm3)'])[0],2)
print(f"The correlation between mouse weight and the average tumor volume is {corr}")
model = st.linregress(capomulin_average['Weight (g)'],capomulin_average['Tumor Volume (mm3)'])

y_values = capomulin_average['Weight (g)']*model[0]+model[1]
plt.scatter(capomulin_average['Weight (g)'],capomulin_average['Tumor Volume (mm3)'])
plt.plot(capomulin_average['Weight (g)'],y_values,color="magenta")
plt.xlabel('Weight (g)')
plt.ylabel('Average Tumor Volume (mm3)')
plt.title("Correlation Weight & Tumor Volume Capomulin Regimen")
plt.show()
# Final Analysis: 
-Evidently, Capomulin had the most mice to complete the study, and all other treatments had a great number of mice deaths during the study, except for Remicane. 
- After analyzing the given data, Capomulin is an effective medication to reduce tumor growth.
- In the data observed, Infubinol had one mouse that had a tumor growth reduction in the study, while the other mice showed a volume increase, proving to be an outlier in the regimen.
- Finally, there was an evident connection between mouse weight and tumor volume. This poses a possibility that mouse weight may affect the drug treatment provided.
