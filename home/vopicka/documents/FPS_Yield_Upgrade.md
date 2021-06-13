# FPS Yield Upgrade

By:

``` text
    Charles E Vopicka
    Senior Biometrician
    Free State Drivers
    Created: 2021.06.13
```

## License

BSD-3-Clause

Copyright 2021 Charles E Vopicka

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.

2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.

3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

- [FPS Yield Upgrade](#fps-yield-upgrade)
  - [License](#license)
  - [Introduction](#introduction)
  - [Configuration](#configuration)
  - [Results](#results)

## Introduction

Yield tables in FPS have always been limiting. The growth model calculates all the same things that a stand gets calculated but then the Yield routine throws all the details away. This causes real trouble when trying to understand the management of yield tables. There are also additional features that can be done simply in a STAND that a YIELD cannot do. One prime example being initial conditions of a YIELD stand can be "faked" but are difficult to track. Solutions to this problem have been offered for a decade and not implemented. This is an attempt to demonstrate that alternate methods can work and provide additional features.

This entire process can be done because Software is stupid. At no point does FPS check if the queried data structures ARE real. So, all that is necessary is to set up a simple shadow of yield structures which the harvest scheduler typically only reads. The key to this working is SIMPLE. We will be replacing all yield structures with queries.  The ultimate solution would be a change to the software which will not happen.

Advantages:

- All stands are treated equally.
- Flagging and assignment can be taken advantage of
- Complex tasks can be applied to yields.
- USER FRIENDLY
- Attention to detail!

## Configuration

1. BACK UP YOUR DATA
2. BACK UP YOUR DATA AGAIN
3. Start with a blank FPS_Zero Database
4. Delete Linked VegPoly (Bad link on most systems)
5. Compact and repair the new database as that was not done before distribution.
6. DELETE Tables:
   - YIELD
   - YIELDENS
   - YLDSPP
   - YLDSRT
7. Create Tables:
   1. Admin_Meta
      - This table needs to be created because FPS cannot have custom fields added to the base tables. This is another attention to detail issue as FPS tends to query * instead of only the fields it needs. This results in the need for FPS to manage fields it has no need to access. This table can be used to add any additional metadata that you may wish to track in the FPS Admin table. You will have to modify custom queries to take full advantage of this.
      - Fields:
        - Std_ID - Long Integer
        - Is_Yield - Yes/No
        - HarvestYr - Integer
        - HarvestRegime - Short Text (255)
8. Create QUERIES:
   - YIELD

        ``` sql
        SELECT
            ADMIN.Basis,
            ADMIN.REGION,
            ADMIN.HAB_GRP,
            CInt([SITE_PHY]) AS Site_Cls,
            [Stand].[RPT_YR]-[msmt_yr] AS RPT_Yr,
            STAND.REGIME,
            STAND.Status,
            STAND.TBR_LBL,
            IIf([Status]=0 And [Admin_meta].[HarvestYr]=([Stand].[RPT_YR]-[msmt_yr]),2,IIf([Status]=1 And [Admin_meta].[HarvestYr]=([Stand].[RPT_YR]-[msmt_yr]),1,0)) AS Flag_Yr,
            STAND.Tot_Age,
            STAND.Trees,
            STAND.QDBH,
            STAND.BASAL,
            STAND.TOP_HT,
            STAND.CCF,
            STAND.RELD,
            STAND.Clump,
            ADMIN.STOCK,
            ADMIN.Origin,
            STAND.CubicTot,
            STAND.CubicGrs,
            STAND.CubicNet,
            STAND.BoardGrs,
            STAND.BoardNet,
            STAND.ValueGrs,
            STAND.CostGrs,
            STAND.ValueNPV,
            STAND.ValueSEV,
            STAND.ValueIRR,
            STAND.APCS,
            STAND.ADIB,
            STAND.MStems,
            STAND.MDbh,
            STAND.MBasal,
            STAND.MAge,
            STAND.MCCF,
            STAND.MRelD,
            STAND.Cords,
            STAND.StemDry,
            STAND.StemWet,
            STAND.BarkDry,
            STAND.CrwnDry,
            STAND.RootDry,
            STAND.BoleDry,
            STAND.BoleWet,
            STAND.VegDry,
            STAND.CarbTree,
            STAND.CarbBole,
            STAND.CO2Tree,
            STAND.CO2Bole
        FROM
            ADMIN
            INNER JOIN STAND ON ADMIN.STD_ID = STAND.STD_ID
            LEFT JOIN Admin_Meta ON ADMIN.STD_ID = Admin_Meta.Std_ID
        WHERE
            (([Stand].[RPT_YR]-[msmt_yr])>=0)
            AND ((Admin_Meta.Is_Yield)=Yes)
        ORDER BY
            ADMIN.Basis,
            ADMIN.REGION,
            ADMIN.HAB_GRP,
            CInt([SITE_PHY]),
            [Stand].[RPT_YR]-[msmt_yr];
        ```

   - YLDENS

        ``` sql
        SELECT
            ADMIN.Basis,
            ADMIN.REGION,
            ADMIN.HAB_GRP,
            ADMIN.SITE_PHY AS Site_Cls,
            [HABDENS].[Rpt_YR]-[Admin].[MSMT_Yr] AS Rpt_Yr,
            HABDENS.Regime,
            HABDENS.Code,
            HABDENS.Age,
            HABDENS.Stems,
            HABDENS.Qdbh,
            HABDENS.Basal,
            HABDENS.CCF,
            HABDENS.RelD
        FROM
            ADMIN
            LEFT JOIN Admin_Meta ON ADMIN.STD_ID = Admin_Meta.Std_ID
            INNER JOIN HABDENS ON ADMIN.STD_ID = HABDENS.Std_ID
        WHERE
            (Admin_Meta.Is_Yield)=Yes;
        ```

   - YLDSPP

        ``` sql
        SELECT
            ADMIN.Basis,
            ADMIN.REGION,
            ADMIN.HAB_GRP,
            CInt([SITE_PHY]) AS Site_Cls,
            PLOTS.SPECIES,
            PLOTS.DBH,
            PLOTS.HEIGHT,
            PLOTS.TREES AS Stems
        FROM
            (ADMIN
            INNER JOIN PLOTS ON (ADMIN.MSMT_YR = PLOTS.MSMT) AND (ADMIN.STD_ID = PLOTS.STD_ID))
            LEFT JOIN Admin_Meta ON ADMIN.STD_ID = Admin_Meta.Std_ID
        WHERE
            (Admin_Meta.Is_Yield)=Yes;
        ```

   - YLDSRT

        ``` sql
        SELECT
            ADMIN.Basis,
            ADMIN.REGION,
            ADMIN.HAB_GRP,
            CInt([SITE_PHY]) AS Site_Cls,
            [standsrt].[Rpt_Yr]-[msmt_yr] AS Rpt_YR,
            STANDSRT.Regime,
            STANDSRT.Flag,
            STANDSRT.Species,
            STANDSRT.Grp,
            STANDSRT.Sort,
            STANDSRT.CubicGrs,
            STANDSRT.CubicNet,
            STANDSRT.BoardGrs,
            STANDSRT.BoardNet,
            STANDSRT.ValueGrs,
            STANDSRT.BoleDry,
            STANDSRT.BoleWet,
            STANDSRT.BarkDry,
            STANDSRT.CarbBole,
            STANDSRT.CO2Bole
        FROM
            (ADMIN
            INNER JOIN STANDSRT ON ADMIN.STD_ID = STANDSRT.Std_ID)
            LEFT JOIN Admin_Meta ON ADMIN.STD_ID = Admin_Meta.Std_ID
        WHERE
            (Admin_Meta.Is_Yield)=Yes;
        ```

9. Create your 'YIELD' stands.
   1. The numbering is not significant however for the sake of simplicity I would suggest using the FPS numbering especially since it will create stant id's outside of most peoples range of numbering.  That system is Region, HabGrp, Site Index.  So 10001090 is (Note the zero padding.)
      - region 10
      - habgrp 1
      - BH site 90.
   2. Populate tables:
      1. ADMIN
         1. VegLbl = GISLbl = 'YELD'
         2. RPT_Yr = MSMT_Yr = Stand.Rpt_Yr - Msmt_Yr
            1. Ya you got to do math and remember this when growing and getting report dates.
            2. Future improvement should include a query to automate this.
            3. The numbers don't matter but the date differences DO
         3. State = XX
         4. Area_Rpt = Area_GIS = Area_Net = 1
            1. This makes it easy with a 1 acre basis
         5. Site_Phy = BH Site Levels
         6. Origin = your Yield Origin Regime
      2. CRUISE
         1. Std_ID = Match Admin!!!
         2. M_Date = 12/31/Year Calculated
         3. MSMT_Yr = Match ADMIN
         4. Cruiser = 'DESK'
         5. BAF_DBH = 999 (meaning data will be a Plot_Area acre representation)
         6. Plot_Area = 1 (1 acre keep life simple)
      3. PLOTS
         1. Std_ID = Match Admin!!!
         2. Plot = 1
         3. Tree = 1 (if you are 'Planting' multiple species just increment this but don't be 0)
         4. Species = Your species DUH
         5. Msmt_Yr = Match ADMIN
         6. TREES = TPA Desired
         7. Height = 1 (Or whatever your planted stock is)
         8. Ht_Code = 1 (Sampled)
         9. Other fields optional.
            1. Crown = 100
            2. Crn_Code = 1 (Sampled)
            3. Defect_[B,M,T] = 0
            4. Def_Code = 1 (Sampled)
10. Flag only your new 'Yield' stands
11. Compile
12. Grow

## Results

This is all preliminary and your results may vary.  If time allows this will need to be finalized.  I have been using this for years and it works great!
