# DNAQuery

DNAQuery is a command line utility to take [logdna](https://logdna.com) archives and load them into [BigQuery](https://cloud.google.com/bigquery/). This allows long term queryable storage of logs in BigQuery which can be considerably most cost effective. In our use case we want real time access to lots of different logs and long term storage of a subset of those logs. This approach has helped us find a balance between access & cost.

## Getting Started

`go get -u github.com/lifechurch/dnaquery`

DNAQuery has been tested on Go v1.9

## Prerequisites

- logdna account must have [Archiving](https://docs.logdna.com/docs/archiving) to S3 enabled
- a [Google Cloud Platform](https://cloud.google.com) account

## Configuring

`cp example.toml dnaquery.toml`

edit dnaquery.toml

All of these settings are currently required.

### AWS
Note: It's recommended that you create a new [IAM user](https://console.aws.amazon.com/iam/home) with `s3:GetObject` permission on the logdna archive previously setup

```
[aws]
Key = "key123123123132" # Key for AWS IAM user
Secret = "secret123123123123" # Secret for AWS IAM user
Bucket = "logs" # name of bucket set in logdna archive setup above
LogPrefix = "a7112abc9d" # each archive file starts with a prefix specific to your logdna account
```

### Storage

```
[storage]
LogDirectory = 'logs' # temp local directory to download logs to and store results, this directory will be created if it doesn't exist
```

### Containers
Note: the term container here is used to be consistent with logdna's terminology
```
[[containers]]
    Name = "production-app"  # name of container
    Regex = '^([\d.]+) - \[([^\]]*)\] - - \[([^\]]*)\] "([^"]*)" (\d+) (\d+) "([^"]*)" "([^"]*)" (\d+) ([\d.]+)  ([\d.:]+) (\d+) ([\d.]+) (\d+)$'  # regex used to pull parts of logs out, currently we dont' use any named capture groups
    TimeGroup = 3 # the number of the capture group (1-based) in above regex that holds the time component of the log
    TimeFormat = "2/Jan/2006:15:04:05 -0700" # the format of the time field using  https://golang.org/pkg/time/#Parse
    [[containers.excludes]]  # an array of tables for exclusions
        Group = 4  # the number of the capture group in above regex to be used in the exclusion check
        Contains = "ping" # excludes logs if the value here is contained in the string in the above capture group
```

### GCP
```
[gcp]  # Google Cloud Project settings
ProjectID = "gcpproj" # name of GCP project
CredentialsFile = "gcp_credentials.json" # path to the credentials file downloaded from GCP
Bucket = "logdna_to_bq" # name of bucket in Cloud Storage to save results for ingestion into BigQuery, bucket will need to be created before first run
Dataset = "logdna" # BigQuery dataset
TemplateTable = "logdna" # currently DNAQuery uses a template table. More details below.
```

#### TemplateTable

Currently the schema for the BigQuery table is specified by creating a table with the schema that matches the regex. This is likely to change in the future as we may need a different schema for each container above.

## Usage

currently dnaquery requires a config file `dnaquery.toml` in the current directory

`dnaquery 2017-11-20`

## Tests

`go test`

## Dependencies

This project uses [dep](https://github.com/golang/de) for dependency management

```
go get -u github.com/golang/dep/cmd/dep
dep ensure -update; dep prune
```

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process
for submitting pull requests.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/lifechurch/dnaquery/tags).

## Authors

* **Chris Vaughn** - *Initial work* - [chrisvaughn](https://github.com/chrisvaughn)

See also the list of [contributors](https://github.com/lifechurch/dnaquery/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
