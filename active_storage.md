# Active Storage

Active Storage

### Configuration

Active Storage uses config/storage.yml file to understand how and where to store uploaded files. If the file doesn't exist, you can create one:

local:

service: Disk

root: <%= Rails.root.join("storage") %>

temporary:\
service: Disk\
root: <%= Rails.root.join("tmp/storage") %>

One stores it in the directory storage, the other stores it in tmp/storage.

#### Use Amazon S3

amazon:

service: S3

access\_key\_id: <%= Rails.application.credentials.dig(:aws, :access\_key\_id) %>

secret\_access\_key: <%= Rails.application.credentials.dig(:aws, :secret\_access\_key) %>

region: us-east-1

bucket: your-bucket-name

After creating or modifying this file, you should also check your environment configuration files (like \`config/environments/development.rb\`) to ensure Active Storage is using the correct service. For example, if you're using local disk storage, you would add this line:

Config/environments/development.rb, to store files in a local folder config.active\_storage.service = :local

Config/environments/test.rb, to store files in a temporary folder

config.active\_storage.service = :temporary

Don’t forget to add /storage to your .gitignore.
