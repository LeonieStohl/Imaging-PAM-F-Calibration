-  This code and all interchangable variables are described in the publication "Standardized Imaging PAM-F for quantifying algal subaerial biomass on building materials" in the journal International Biodeterioration and Biodegradation 
-  The variables in this code are tailored to a dataset available here: DOI 10.5281/zenodo.17184214
-  For the use on own data, the code must be adapted to the own Datastructure (e.g. structure of an Excelfile) and the own experimental variables.

Structure of the Script:

1.) Loads an Excel workbook containing multiple sheets, each representing a different experimental setting. Extracts material, setting, replicate and F0 signal
  ADJUST DATA IMPORT TO THE STRUCTURE OF THE EXCEL FILE

2.) Filtering each replicate for valid signals to determine the edge cases. This is needed because of the high x-range and the fact that the majority of the signal mesured are clumped in a comparetively short x - range. Without this filtering, the linear fitting does not correctly detect lower detection limits 
Input: df_filtered_replicates columns: [Material, Replicate, Concentration, Signal] 
Output: df_signal_averages columns: [Material, Replicate, Concentration, Signal] 
• If invalid values within the first 4 values, all previous values are also invalid and the current value is the new start value. 
• If invalid within the last 4 values, mark the current one and all subsequent ones invalid 
• NaN for signal that are 0 or dont meet the threshhold increase of 5% • Check for data quality within a replicate
  ADJUST THE RANGE OF INITIAL (START) AND TERMINAL (END) AND ADJUST SIGNAL INCREASE FILTER (Si)
  
3.) Calculating signal averages from the filtered replicate data 
Input: df_filtered_replicates columns: [Material, Replicate, Concentration, Signal] 
Output: df_signal_averages_raw columns: [Material, Setting, Concentration, Signal_Average, Signal_Error] 

Check whether all 4 replicate values are valid (not NaN) --> Check for data quality within all four replicates/group of setting + material

4.) Cutting dataframe entries 
Input: df_signal_averages_raw columns: [Material, Setting, Concentration, Signal_Average, Signal_Error] 
Output: df_signal_averages columns: [Material, Setting, Concentration, Signal_Average, Signal_Error] 
• Minimum length of subsequent valid F0 signals (=calibration range) is set to 4 
• Keep all valid groups and cut the rest
  ADJUST MINIMUM NUMBER OF CONSECUTIVE, VALID DATAPOINTS (nmin). 

5.) Linear fits + rating of fits with Sliding Window of 4 - 9 
Input: df_signal_averages columns: [Material, Setting, Concentration, Signal_Average, Signal_Error] 
Output: df_fits columns: [Material, Setting, Start_Concentration, End_Concentration, Range_Width, R2_Value, Slope, Intercept, Rating]
• Linar fitting: Weighted Least Squares (WLS) regression • Error Calculation: Inverse-variance weighting
• Residuals filter: 20% (=rmax)
• R2 min: 0.8 (=R2min)
• rating = 0.5 (=a) * norm_r2 + 0.5 (=b) * norm_range - 100 * residual_mean 
ADJUST FILTERING AND WEIGHTING PARAMETERS (nmin, nmax, rmax, R2min, a, b)

6.) Choosing the best linear fit for each Setting + Material combination with the previously calculated score. Cuts away all linear fits that were not chosen and drastically reduces the data frame 
Input: df_fits columns: [Material, Setting, Start_Concentration, End_Concentration, Range_Width, R2_Value, Slope, Intercept, Rating]
Output: df_best_fits columns: [Material, Setting, Start_Concentration, End_Concentration, Range_Width, R2_Value, Slope, Intercept, Rating]

7.) Choosing the setting for each Material. Cuts away all settings that were not chosen and drastically reduces the data frame 
Input: df_best_fits columns: [Material, Setting, Start_Concentration, End_Concentration, Range_Width, R2_Value, Slope, Intercept, Rating] 
Output: df_best_settings columns: [Material, Setting, Start_Concentration, End_Concentration, Range_Width, R2_Value, Slope, Intercept, Rating]

8.) Data Export -> Excel

9.) Plotting
