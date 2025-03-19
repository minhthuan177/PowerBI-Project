# PowerBI-Project
Báo cáo kết quả kinh doanh của công ty Plant Co., so sánh với cùng kỳ năm trước.
Dataset: Plant_DTS.xls

Công thức:

Bảng dim_date:
  dim_date = CALENDAR(DATE(2022,01,01), DATE(2024,12,31)) 
  
  Inpast = 
  VAR lastsalesdate = MAX(fact_sales[date_time])
  VAR lastsalesdatePY = EDATE(lastsalesdate, -12)
  RETURN
  dim_date[Date] <= lastsalesdatePY

Thêm bảng Slc_values để làm slicer

Bảng _measure:
  Sales = SUM(fact_sales[sales_USD])
  Quantity = SUM(fact_sales[quantity])
  COGs = SUM(fact_sales[COGS_USD]) 
  Gross Profit = [Sales] - [COGs]
  
Lũy kế cùng kỳ năm trước Prior YTD:
  PYTD_Sales = CALCULATE([Sales], SAMEPERIODLASTYEAR(dim_date[Date]), dim_date[Inpast] = TRUE)
  PYTD_Quantity = CALCULATE([Quantity], SAMEPERIODLASTYEAR(dim_date[Date]), dim_date[Inpast] = TRUE)
  PYTD_GrossProfit = CALCULATE([Gross Profit], SAMEPERIODLASTYEAR(dim_date[Date]), dim_date[Inpast] = TRUE)

Lũy kế năm nay YTD:
  YTD_Sales = TOTALYTD([Sales], fact_sales[date_time])
  YTD_Quantity = TOTALYTD([Quantity], fact_sales[date_time])
  YTD_GrossProfit = TOTALYTD([Gross Profit], fact_sales[date_time])

Switch giúp slicer điều chỉnh đến toàn bộ các giá trị:
  S_PYTD = 
  VAR selected_value = SELECTEDVALUE(slc_values[Value])
  VAR result = SWITCH(selected_value,
    "Sales", [PYTD_Sales],
    "Quantity", [PYTD_Quantity],
    "Gross Profit", [PYTD_GrossProfit],
    BLANK()
  )
  RETURN result

  S_YTD = 
  VAR selected_value = SELECTEDVALUE(slc_values[Value])
  VAR result = SWITCH(selected_value,
    "Sales", [YTD_Sales],
    "Quantity", [YTD_Quantity],
    "Gross Profit", [YTD_GrossProfit],
  BLANK()
  )
  RETURN result
So sánh YTD và PYTD:
  YTD vs PYTD = [S_YTD] - [S_PYTD]
  
