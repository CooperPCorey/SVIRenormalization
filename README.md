# SVIRenormalization
Calculating SVI to a unique study area
# =========================================================
# 0. PACKAGES
# =========================================================

library(tidycensus)
library(dplyr)
library(tidyr)

# =========================================================
# 1. CENSUS API KEY
# =========================================================

census_api_key("YOUR_API_KEY_HERE", install = TRUE)

# =========================================================
# 2. VARIABLE LIST (FULL CDC SVI STRUCTURE)
# =========================================================

vars <- c(
  # -------------------------
  # Population
  # -------------------------
  total_pop = "B01003_001",
  total_hh = "B25003_001",

  # -------------------------
  # Theme 1: Socioeconomic
  # -------------------------
  poverty = "B17010_002",
  unemployed = "B23025_005",
  labor_force = "B23025_003",

  nohs = "B15003_017",
  pop25 = "B15003_001",

  uninsur = "B27001_005",

  owner_30_50 = "B25106_005",
  owner_50_plus = "B25106_006",
  renter_30_50 = "B25106_011",
  renter_50_plus = "B25106_012",

  # -------------------------
  # Theme 2: Household
  # -------------------------
  age65 = "B01001_020",
  age17 = "B01001_003",
  disability = "B18101_001",
  single_parent = "B11003_010",
  lim_eng = "B16004_001",

  # -------------------------
  # Theme 3: Race/Ethnicity
  # -------------------------
  race_total = "B03002_001",
  white_nh = "B03002_003",

  # -------------------------
  # Theme 4: Housing/Transport
  # -------------------------
  unit_2_4 = "B25024_002",
  unit_5_9 = "B25024_003",
  unit_10_19 = "B25024_004",
  unit_20_plus = "B25024_005",

  crowd = "B25014_005",
  noveh = "B25044_003",
  groupq = "B26001_001"
)

# =========================================================
# 3. PULL ACS DATA
# =========================================================

raw_acs <- get_acs(
  geography = "county",
  variables = vars,
  year = 2024,
  survey = "acs5",
  geometry = FALSE
)

# =========================================================
# 4. RESHAPE WIDE FORMAT
# =========================================================

acs_wide <- raw_acs %>%
  select(GEOID, NAME, variable, estimate) %>%
  pivot_wider(
    names_from = variable,
    values_from = estimate
  )

# =========================================================
# 5. CREATE RATES
# =========================================================

svi_df <- acs_wide %>%
  mutate(

    # -------------------------
    # Theme 1
    # -------------------------
    pov_rate = poverty / total_pop,
    unemp_rate = unemployed / labor_force,
    nohs_rate = nohs / pop25,
    unins_rate = uninsur / total_pop,

    hburd_rate =
      (owner_30_50 + owner_50_plus +
       renter_30_50 + renter_50_plus) / total_hh,

    # -------------------------
    # Theme 2
    # -------------------------
    age65_rate = age65 / total_pop,
    age17_rate = age17 / total_pop,
    disability_rate = disability / total_pop,
    single_parent_rate = single_parent / total_pop,
    limeng_rate = lim_eng / total_pop,

    # -------------------------
    # Theme 3
    # -------------------------
    minority_rate = 1 - (white_nh / race_total),

    # -------------------------
    # Theme 4
    # -------------------------
    multi_unit_rate =
      (unit_2_4 + unit_5_9 + unit_10_19 + unit_20_plus) / total_pop,

    crowd_rate = crowd / total_pop,
    noveh_rate = noveh / total_pop,
    groupq_rate = groupq / total_pop
  )

# =========================================================
# 6. BUILD THEMES
# =========================================================

svi_df <- svi_df %>%
  mutate(

    theme1 = pov_rate +
              unemp_rate +
              nohs_rate +
              unins_rate +
              hburd_rate,

    theme2 = age17_rate +
              age65_rate +
              disability_rate +
              single_parent_rate +
              limeng_rate,

    theme3 = minority_rate,

    theme4 = multi_unit_rate +
              crowd_rate +
              noveh_rate +
              groupq_rate
  )

# =========================================================
# 7. CDC NORMALIZATION (PERCENT RANKS)
# =========================================================

svi_df <- svi_df %>%
  mutate(

    theme1_r = percent_rank(theme1),
    theme2_r = percent_rank(theme2),
    theme3_r = percent_rank(theme3),
    theme4_r = percent_rank(theme4),

    svi_index = theme1_r + theme2_r + theme3_r + theme4_r
  )

# =========================================================
# 8. FINAL OUTPUT
# =========================================================

final_svi <- svi_df %>%
  select(
    GEOID, NAME,

    theme1, theme2, theme3, theme4,
    theme1_r, theme2_r, theme3_r, theme4_r,

    svi_index
  )
