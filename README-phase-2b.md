0. The goal of phase 2b is to perform benchmarking/scalability tests of sample three-tier lakehouse solution.

1. In main.tf, change machine_type at:

```
module "dataproc" {
  depends_on   = [module.vpc]
  source       = "github.com/bdg-tbd/tbd-workshop-1.git?ref=v1.0.36/modules/dataproc"
  project_name = var.project_name
  region       = var.region
  subnet       = module.vpc.subnets[local.notebook_subnet_id].id
  machine_type = "e2-standard-2"
}
```

and subsititute "e2-standard-2" with "e2-standard-4".

2. If needed request to increase cpu quotas (e.g. to 30 CPUs): 
https://console.cloud.google.com/apis/api/compute.googleapis.com/quotas?project=tbd-2023z-9918

3. Using tbd-tpc-di notebook perform dbt run with different number of executors, i.e., 1, 2, and 5, by changing:
```
 "spark.executor.instances": "2"
```

in profiles.yml.

4. In the notebook, collect console output from dbt run, then parse it and retrieve total execution time and execution times of processing each model. Save the results from each number of executors. 

5. Analyze the performance and scalability of execution times of each model. Visualize and discucss the final results.

W celu analizy wpływu liczby executorów na wydajność przetwarzania modeli przez dbt, przeprowadziliśmy testy z użyciem 1, 2 oraz 4 executorów. Poniższa tabela przedstawia czas przetwarzania modelu w sekundach w zależności od liczby użytych executorów. Różnica procentowa wyraża, o jaki procent udało się zredukować czas wykonania dzięki użyciu 4 executorów zamiast 1.


|  Model                                    |   1 Executor |   2 Executors |   4 Executors |   Róznica procentowa |
|:---------------------------------------|-------------:|--------------:|--------------:|------------------------:|
| demo_silver.daily_market               |      2268.97 |       1018.83 |        440.6  |              -80.5815   |
| demo_silver.trades_history             |       486.4  |        264.39 |        130.65 |              -73.1394   |
| demo_silver.trades                     |       403.52 |        220.48 |        113.23 |              -71.9394   |
| demo_silver.holdings_history           |       154.04 |         82.91 |         43.95 |              -71.4684   |
| demo_silver.watches                    |       243.45 |        123.55 |         69.74 |              -71.3535   |
| demo_bronze.brokerage_trade_history    |        57.71 |         38.13 |         17.47 |              -69.728    |
| demo_gold.dim_trade                    |       313.04 |        175.3  |         99.49 |              -68.2181   |
| demo_bronze.finwire_financial          |        83.71 |         42.67 |         28.43 |              -66.0375   |
| demo_silver.watches_history            |       160.5  |         91.78 |         58.73 |              -63.4081   |
| demo_bronze.brokerage_daily_market     |       142.14 |         77.64 |         52.05 |              -63.3812   |
| demo_bronze.brokerage_trade            |        66.01 |         40.51 |         24.33 |              -63.1419   |
| demo_bronze.brokerage_watch_history    |        70.32 |         47.52 |         29.37 |              -58.2338   |
| demo_silver.financials                 |       101.55 |         57.84 |         42.78 |              -57.873    |
| demo_bronze.syndicated_prospect        |        10.18 |          5.64 |          4.61 |              -54.7151   |
| demo_silver.cash_transactions          |        72.49 |         45.15 |         33.84 |              -53.3177   |
| demo_gold.fact_watches                 |       494.97 |        272.34 |        232.03 |              -53.1224   |
| demo_bronze.brokerage_holding_history  |        13.88 |          9.61 |          6.98 |              -49.7118   |
| demo_bronze.brokerage_cash_transaction |        42.29 |         29.4  |         23.09 |              -45.4008   |
| demo_gold.fact_holdings                |      1428.01 |        730.16 |        784.03 |              -45.0963   |
| demo_gold.fact_trade                   |       793.56 |        536.08 |        474.97 |              -40.1469   |
| demo_silver.employees                  |         4.12 |          3.71 |          2.62 |              -36.4078   |
| demo_gold.fact_cash_balances           |       250.95 |        247.74 |        160.58 |              -36.0112   |
| demo_gold.dim_customer                 |        18.31 |         21.65 |         11.72 |              -35.9913   |
| demo_bronze.reference_status_type      |         1.23 |          0.74 |          0.79 |              -35.7724   |
| demo_bronze.finwire_company            |         2.38 |          2.23 |          1.56 |              -34.4538   |
| demo_bronze.finwire_security           |         2.61 |          2.8  |          1.74 |              -33.3333   |
| demo_silver.customers                  |         9.89 |         10.94 |          6.76 |              -31.6481   |
| demo_silver.accounts                   |        13.71 |         13.35 |          9.52 |              -30.5616   |
| demo_gold.dim_broker                   |         4.6  |          5.21 |          3.32 |              -27.8261   |
| demo_silver.companies                  |         6.55 |          6.07 |          5.1  |              -22.1374   |
| demo_bronze.crm_customer_mgmt          |         8.67 |          8.16 |          6.76 |              -22.03     |
| demo_bronze.hr_employee                |         3.06 |          3.24 |          2.4  |              -21.5686   |
| demo_gold.dim_account                  |        13.47 |         12.75 |         10.75 |              -20.193    |
| demo_silver.date                       |         1.39 |          1.33 |          1.14 |              -17.9856   |
| demo_bronze.reference_industry         |         0.92 |          0.76 |          0.78 |              -15.2174   |
| demo_bronze.reference_date             |         1.22 |          1.01 |          1.08 |              -11.4754   |
| demo_silver.securities                 |         4.61 |          5.04 |          4.21 |               -8.67679  |
| demo_bronze.reference_tax_rate         |         0.78 |          0.7  |          0.72 |               -7.69231  |
| demo_gold.dim_company                  |         2.41 |          3.17 |          2.27 |               -5.80913  |
| demo_gold.dim_security                 |         4.08 |          3.87 |          4    |               -1.96078  |
| demo_gold.dim_date                     |         1.29 |          1.4  |          1.3  |                0.775194 |
| demo_bronze.reference_trade_type       |         0.78 |          0.76 |          1.07 |               37.1795   |
| demo_gold.fact_cash_transactions       |        81.51 |         96.45 |        210.63 |              158.41     |


W zdecydowanej większości przypadków użycie większej liczby executorów znacząco przyspiesza przetwarzanie modeli, dla modeli na szczycie tabeli uzyskano niemal czterokrotnie lepsze czasy. Niemniej jednak, w przypadku modeli demo_gold.dim_date, demo_bronze.reference_trade_type oraz demo_gold.fact_cash_transactions zaobserwowano spowolnienie. Dwa pierwsze modele mają bardzo krótki czas wykonania, więc rozsądne byłoby ponownce przeprowadzenie testów, aby zminimalizować szum. Natomiast w przypadku demo_gold.fact_cash_transactions spowolnienie jest wyraźnie zauważalne.

W rezultacie, łączny czas przetwarzania modeli ulega znaczącej redukcji wraz ze wzrostem liczby używanych executorów. Niemniej jednak, zebrane dane pozwalają zaobserwować, że skalowanie to nie przebiega w sposób liniowy w stosunku do wzrostu liczby executorów. 

![image](https://github.com/user-attachments/assets/f94dbfa6-9b47-4e74-86a5-03134a7a36bb)

