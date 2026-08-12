Hello! There are two code methodologies presented here, the first is a code that will renormalize the CDC's 2022 SVI to a specific study area. 
This is an important step of analyzing social vulnerability, as counties are ranked by major themes, and their full SVI score is on a scale of 0-1. 
So, if you remove areas from the original dataset, these ranks and scores will not be correct. 

My study area, the eastern half of the U.S., is input into "Step 1" where you include the states you want. Don't forget to change these lines if you have a different study area. 
This code requires you to have already downloaded the full SVI dataset (CSV), which is available here: https://www.atsdr.cdc.gov/place-health/php/svi/svi-data-documentation-download.html
Note that this code will ONLY WORK for the 2022 SVI, as the specific variables used by the CDC changes with every new dataset. 

The second code provided here is a manual calculation of the "2024 SVI" using the CDC's methodology for 2022. This code pulls the necessary variables from the Census' 2024 ACS dataset.
You will need an API key to run this code. It is free and easy to obtain (see README.API.KEY for instructions). You will only need to write in your key into RStudio ONCE.
Again, this code is written for a study area consisting only of the eastern states, so if you have a different study area you will need to change the lines in "Step 3"
This code appears to be working, and the results seem sound, but still: USE WITH CAUTION! I am one person unaffiliated with the CDC, and there is no 2024 dataset to compare these results with

If you have any questions, feel free to reach out! 
