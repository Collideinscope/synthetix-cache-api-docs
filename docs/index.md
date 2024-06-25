# Synthetix Cache API Documentation

## Metrics and Transformations

| Metric          | Source                                 | Query Frequency       | Model (Schema) | Transformations from Source                           | Transformations to Endpoint                  |
|-----------------|----------------------------------------|-----------------------|----------------|------------------------------------------------------|---------------------------------------------|
| **TVL**         | base_mainnet.core_vault_collateral     | SELECT ts, pool_id, collateral_type, amount, collateral_value | Model: Yes, Query: No | ts = calendar hour, block_ts = original model ts from block | Add chain                                   |
|                 |                                        |                       | **tvl**        |                                                      |                                             |
|                 |                                        |                       | `id SERIAL PRIMARY KEY,` | 1 amount per calendar hour (highest from hour) |                                             |
|                 |                                        |                       | `created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | |                                             |
|                 |                                        |                       | `updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | |                                             |
|                 |                                        |                       | `ts TIMESTAMP WITH TIME ZONE NOT NULL,` | |                                             |
|                 |                                        |                       | `chain TEXT NOT NULL,` | |                                             |
|                 |                                        |                       | `pool_id INTEGER NOT NULL,` | |                                             |
|                 |                                        |                       | `collateral_type TEXT NOT NULL,` | |                                             |
|                 |                                        |                       | `amount NUMERIC(30, 10) NOT NULL,` | |                                             |
|                 |                                        |                       | `collateral_value NUMERIC(30, 10) NOT NULL,` | |                                             |
|                 |                                        |                       | `block_ts TIMESTAMP WITH TIME ZONE NOT NULL,` | |                                             |
|                 |                                        |                       | `block_number INTEGER NOT NULL,` | |                                             |
|                 |                                        |                       | `contract_address TEXT NOT NULL,` | |                                             |
|                 |                                        |                       | `UNIQUE (chain, ts, pool_id, collateral_type)` | |                                             |
| **Total Delegations (staked)** | base_mainnet.fct_core_pool_delegation | SELECT * FROM fct_core_pool_delegation ORDER BY ts DESC LIMIT 1; | Model: Yes, Query: No | ts = calendar hour, block_ts = original model ts from block | Add chain |
|                 |                                        | SELECT * FROM fct_core_pool_delegation ORDER BY ts DESC; | **core_delegations** | 1 amount per calendar hour (highest from hour) | |
|                 |                                        |                       | `id SERIAL PRIMARY KEY,` | | |
|                 |                                        |                       | `created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        |                       | `updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        |                       | `ts TIMESTAMP WITH TIME ZONE NOT NULL,` | | |
|                 |                                        |                       | `chain TEXT NOT NULL,` | | |
|                 |                                        |                       | `pool_id INTEGER NOT NULL,` | | |
|                 |                                        |                       | `collateral_type TEXT NOT NULL,` | | |
|                 |                                        |                       | `amount_delegated NUMERIC NOT NULL,` | | |
|                 |                                        |                       | `block_ts TIMESTAMP WITH TIME ZONE NOT NULL,` | | |
|                 |                                        |                       | `UNIQUE (chain, ts, pool_id, collateral_type)` | | |
| **APY**         | base_mainnet.fct_core_apr              | SELECT ts, pool_id, collateral_type, collateral_value, apy_24h, apy_7d, apy_28d | Model: None, Query: Yes | Add chain, timeframes for each model timeframe | |
|                 |                                        | ORDER BY ts DESC LIMIT 1 | **apy**        | | |
|                 |                                        | SELECT ts, pool_id, collateral_type, collateral_value, apy_24h, apy_7d, apy_28d | `id SERIAL PRIMARY KEY,` | | |
|                 |                                        | ORDER BY ts DESC;      | `created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        |                       | `updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        |                       | `ts TIMESTAMP WITH TIME ZONE NOT NULL,` | | |
|                 |                                        |                       | `chain TEXT NOT NULL,` | | |
|                 |                                        |                       | `pool_id INTEGER NOT NULL,` | | |
|                 |                                        |                       | `collateral_type TEXT NOT NULL,` | | |
|                 |                                        |                       | `collateral_value NUMERIC NOT NULL,` | | |
|                 |                                        |                       | `apy_24h NUMERIC NOT NULL,` | | |
|                 |                                        |                       | `apy_7d NUMERIC,` | | |
|                 |                                        |                       | `apy_28d NUMERIC,` | | |
|                 |                                        |                       | `UNIQUE (chain, ts, pool_id, collateral_type)` | | |
| **Pool Rewards** | base_mainnet.fct_pool_rewards_hourly | SELECT pool_id, SUM(rewards_usd) AS total_rewards_usd | Model: None, Query: No | None, original ts by calendar hour | |
|                 |                                        | FROM base_mainnet.fct_pool_rewards_hourly | **pool_rewards** | | |
|                 |                                        | GROUP BY pool_id ORDER BY pool_id | `id SERIAL PRIMARY KEY,` | | |
|                 |                                        |                       | `created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        | SELECT ts, pool_id, SUM(rewards_usd) AS total_rewards_usd | `updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        |                       | `ts TIMESTAMP WITH TIME ZONE NOT NULL,` | | |
|                 |                                        |                       | `chain TEXT NOT NULL,` | | |
|                 |                                        |                       | `pool_id INTEGER NOT NULL,` | | |
|                 |                                        |                       | `collateral_type TEXT NOT NULL,` | | |
|                 |                                        |                       | `rewards_usd NUMERIC NOT NULL,` | | |
|                 |                                        |                       | `UNIQUE (chain, ts, pool_id, collateral_type)` | | |
| **Staker Count** | base_mainnet.fct_core_account_delegation | SELECT pool_id, COUNT(DISTINCT account_id) AS staker_count | Model: None, Query: No | Keep original ts since not a time series, no transformations | |
|                 |                                        | FROM base_mainnet.fct_core_account_delegation | **core_account_delegations** | | |
|                 |                                        | GROUP BY pool_id ORDER BY pool_id | `id SERIAL PRIMARY KEY,` | | |
|                 |                                        |                       | `created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        | SELECT DATE_TRUNC('day', ts) AS day, pool_id, COUNT(DISTINCT account_id) AS staker_count | `updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,` | | |
|                 |                                        |                       | `ts TIMESTAMP WITH TIME ZONE NOT NULL,` | | |
|                 |                                        |                       | `chain TEXT NOT NULL,` | | |
|                 |                                        |                       | `account_id TEXT NOT NULL,` | | |
|                 |                                        |                       | `pool_id INTEGER NOT NULL,` | | |
|                 |                                        |                       | `collateral_type TEXT NOT NULL,` | | |
|                 |                                        |                       | `amount_delegated NUMERIC(30, 10) NOT NULL,` | | |
|                 |                                        |                       | `UNIQUE (chain, account_id, pool_id, collateral_type)` | | |

