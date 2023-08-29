# Description

This role configures the [Lido Validator Ejector](https://github.com/lidofinance/validator-ejector)

This is a daemon which monitors [`ValidatorsExitBusOracle`](https://github.com/lidofinance/lido-dao/blob/feature/shapella-upgrade/contracts/0.8.9/oracle/ValidatorsExitBusOracle.sol) events on a target network and broadcasts pre-generated validator exit messages when necessary (as users of the LIDO protocol request withdrawals).

On start-up, it loads exit messages from a specified folder in form of individual `.json` files and validates their format, structure and signature. Then, it fetches log events from the most recent `ejector_blocks_preload` blocks, checks whether any exits should be performed immediately and then enteres a polling loop that checks for new events every `ejector_blocks_loop` blocks.

For further details, please refer to the [`Validator Ejector` Documentation](https://hackmd.io/@lido/BJvy7eWln).

## Role Implementation Details

The software runs as a docker container from [`lidofinance/validator-ejector`](https://hub.docker.com/r/lidofinance/validator-ejector).

Docker compose file template is in [templates/docker-compose.yml.j2](./templates/docker-compose.yml.j2).

## Environment Variables

Options are configured via environment variables. They are defined at [defaults/main.yml](./defaults/main.yml).

> **Warning:** You cannot use `ejector_validator_exit_webhook` or the `ejector_messages_location` at the same time. Only one of the two variables must be set

| Variable                           | Required | Description                                                                                                                                                                                                                                             |
| --------------------------         | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ejector_execution_node             | Yes      | Ethereum Execution Node endpoint                                                                                                                                                                                                                        |
| ejector_consensus_node             | Yes      | Ethereum Consensus Node endpoint                                                                                                                                                                                                                        |
| ejector_locator_address            | Yes      | Address of the Locator contract [Goerli](https://docs.lido.fi/deployed-contracts/goerli/) / [Mainnet](https://docs.lido.fi/deployed-contracts/)                                                                                                         |
| ejector_staking_module_id          | Yes      | Staking Module ID for which operator ID is set, currently only one exists - ([NodeOperatorsRegistry](https://github.com/lidofinance/lido-dao#contracts)) with id `1`                                                                                    |
| ejector_operator_id                | Yes      | Operator ID in the Node Operators registry, easiest to get from Operators UI: [Goerli](https://operators.testnet.fi)/[Mainnet](https://operators.lido.fi)                                                                                               |
| ejector_messages_location          | No       | Local folder or external storage bucket url to load json exit message files from. Required if you are using exit messages mode                                                                                                                          |
| ejector_validator_exit_webhook     | No       | POST validator info to an endpoint instead of sending out an exit message in order to initiate an exit. Required if you are using webhook mode                                                                                                          |
| ejector_oracle_addresses_allowlist | Yes      | Allowed Oracle addresses to accept transactions from [Goerli](https://testnet.testnet.fi/#/lido-testnet-prater/0x24d8451bc07e7af4ba94f69acdd9ad3c6579d9fb/) / [Mainnet](https://mainnet.lido.fi/#/lido-dao/0x442af784a788a5bd6f42a01ebe9f287a871243fb/) |
| ejector_messages_password          | No       | Password to decrypt encrypted exit messages with. Needed only if the exit messages are encrypted                                                                                                                                                        |
| ejector_messages_password_file     | No       | Path to a file with password inside to decrypt exit messages with. Needed only if you have encrypted exit messages. If used, MESSAGES_PASSWORD (not MESSAGES_PASSWORD_FILE) needs to be added to LOGGER_SECRETS in order to be sanitized                |
| ejector_blocks_preload             | No       | Amount of blocks to load events from on start. Increase if daemon was not running for some time. Defaults to a week of blocks                                                                                                                           |
| ejector_blocks_loop                | No       | Amount of blocks to load events from on every poll. Defaults to 3 hours of blocks                                                                                                                                                                       |
| ejector_job_interval               | No       | Time interval in milliseconds to run checks. Defaults to time of 1 epoch                                                                                                                                                                                |
| ejector_http_port                  | No       | Port to serve metrics and health check on                                                                                                                                                                                                               |
| ejector_run_metrics                | No       | Enable metrics endpoint                                                                                                                                                                                                                                 |
| ejector_run_health_check           | No       | Enable health check endpoint                                                                                                                                                                                                                            |
| ejector_logger_level               | No       | Severity level from which to start showing errors eg info will hide debug messages                                                                                                                                                                      |
| ejector_logger_format              | No       | Simple or JSON log output: simple/json                                                                                                                                                                                                                  |
| ejector_logger_secrets             | No       | JSON string array of either env var keys to sanitize in logs or exact values                                                                                                                                                                            |
| ejector_dry_run                    | No       | Run the service without actually sending out exit messages                                                                                                                                                                                              |

Messages can also be loaded from remote storages: AWS S3 and Google Cloud Storage.

Simply set a url with an appropriate protocol in `ejector_messages_location`:

- `s3://` for S3
- `gs://` for GCS

## Requirements

Hardware

- 2-core CPU
- 1GB RAM
