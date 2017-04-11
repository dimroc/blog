---
layout: post
title: "rake ml:learn - AWS Machine Learning through rake"
date: "Tue Apr 11 12:07:59 -0400 2017"
tags: aws machine-learning ruby etl
---

Automating the creation of your machine learning (ML) model can allow your
services to evolve over time, automatically.

Ideally, we would have a model that would use Stochastic Gradient Descent
and would learn per query. But when you want a low maintenance solution such
as [AWS ML](https://aws.amazon.com/machine-learning/), that isn't an option,
since AWS ML models are immutable by design.

You can, however, retrain a new model and then flip the switch so queries go
to the new endpoint.

Achieving this requires the class **ETL** or Extract, Transform, Load. This article
will be focusing on that last step **Load**. Before we talk about load, let's
set up a simple extract and transform so we all have context. Simple meaning
no multiple datastores or clusters.

## Extract

- Specific to your business logic
- Can be skipped in this simple example and fed directly into the transform
- More complicated examples might require getting data dumps from multiple databases
  or other teams

```ruby
module Machine
  class Extractor
    def self.perform
      CSV.open(Rails.root.join("tmp", "extracted_data.csv"), "w") do |csv|
        ImportantData.find_each do |data|
          csv << data.to_csv
        end
      end
    end
  end
end
```

## Transform

- Critical step involves converting your raw data into features to be ingested
  by AWS Machine Learning models and evaluations
- What those features are depends on your data and your machine learning model,
  whether it be simple linear regression or a deep neural network

```ruby
module Machine
  class Transformer
    def self.perform(filename = Rails.root.join("tmp", "extracted_data.csv"))
      Dir.mkdir('tmp') unless File.exists?('tmp')
      AwsMlTransformer.new(filename, "tmp/intermediary.csv").write
      FeatureExpanderWriter.new("tmp/intermediary.csv", "tmp/features.csv").write
    end
  end
end
```

## Load

The good stuff. Look at the steps listed in the method calls below. We'll
be going through each of them.

```ruby
module Machine
  class Loader
    def self.perform(filename=Rails.root.join("tmp","features.csv"))
      instance = new
      instance.upload_data_to_s3(filename)
      instance.create_data_sources
      instance.create_model
      instance.create_evaluation
      instance.create_realtime_endpoint
    end
```

### Upload Data to S3

- AWS ML needs access to your S3 bucket

### Create Data Sources

- Hardest step
- Complete data source
- Data Rearragement
- Compute statistics
- Data Schema

### Create Model From Data Source

- Just run that command

### Create Evaluation

- Show F1 Heat Map

### Create Realtime Endpoint

- Expose it to your service!
