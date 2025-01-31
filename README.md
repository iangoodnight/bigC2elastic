![bigc2elastic](https://github.com/iangoodnight/bigc2elastic/blob/main/bigc2elastic.png)

# bigc2elastic

A lightweight CLI utility designed to pull product and category data from
BigCommerce stores and push that data back up to an elasticsearch instance.

# What it does

Running this utility will:

- Query BigCommerce for categories
- Check returned categories for `is_visible`
- Attempt to delete any hidden categories from elasticsearch (if they exist)
- POST the remaining categories to elasticsearch, updating them if changed
- Query BigCommerce for brands
- POST brands to elasticsearch, updating them if changed
- Query BigCommerce for products
- Check returned products for `is_visible`
- Map categories and brands to products
- Attempt to delete any hidden products from elasticsearch (if they exist)
- POST the remaining products to elasticsearch, updating them if changed
- Output all steps and processes colorfully to the terminal

# CLI Flags

Optional flags include:

| Flag | Example | Effect |
| ---- | ------- | ------ |
| init | `bigc2elastic --init` | CLI to set necessary ENV variables |
| drop | `bigc2elastic --drop` | Drops existing documents first (reset) |
| products-only | `bigc2elastic --products-only` | Syncs products only |
| no-categories | `bigc2elastic --no-categories` | Skips category sync |
| no-brands | `bigc2elastic --no-brands` | Skips brand sync |

If ENV variables are missing or not configured, bigc2elastic runs `bigc2elastic
--init` whether or not the `--init` flag is included.

Running the CLI with the `--products-only` flag still queries BigCommerce for
categories and brands to decorate product documents with their category and
brand names, but it will not drop or update category or brand documents within
elasticsearch.

Optionally, just the brand documents or just the category documents may be
omitted with the appropriate flags (ie: `--no-categories`, `--no-brands`).

Running the CLI with the `--drop` flag will drop all existing documents from
elasticsearch before beginning the updates (effectively starting clean).

Flags may be combined if necessary (ie: `bigc2elastic --drop
--no-categories`).

## Document Schema

| Field | Type | Product | Category | Brand |
| ----- | ---- | ------- | -------- | ----- |
| brand | `<string>` | ✓ | ✗ | ✗ |
| bucket | `<string>` | `'product'` | `'category'` | `'brand'` |
| calculated_price | `<number>` | ✓ | ✗ | ✗ |
| categories | [`<string>,...]` | ✓ | ✗ | ✗ |
| date_created | `<string>` | ✓ | ✗ | ✗ |
| date_modified | `<string>` | ✓ | ✗ | ✗ |
| description | `<string>` | ✓ | ✓ | ✗ |
| id | `<string>` | `bc2eP<id>` | `bc2eC<id>` | `bc2eB<id>` |
| mpn | `<string>` | ✓ | ✗ | ✗ |
| name | `<string>` | ✓ | ✓ | ✓ |
| page_title | `<string>` | ✓ | ✓ | ✓ |
| price | `<number>` | ✓ | ✗ | ✗ |
| search_keywords | `<string>` | ✓ | ✓ | ✓ |
| sort_order | `<number>` | ✓ | ✓ | ✗ |
| total_sold | `<number>` | ✓ | ✗ | ✗ |
| upc | `<string>` | ✓ | ✗ | ✗ |
| view_count | `<number>` | ✓ | ✓ | ✗ |
| url | `<string>` | ✓ | ✓ | ✓ |

## Setup

1. First install nvm with `curl
   https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash`
   on Debian-based systems, update your environment settings with `source
   ~/.profile`, and install node with `nvm install latest` (alternatively,
   check out https://nodejs.org/en/download/ for other installation options).
2. Install globally via npm with `npm install -g @goodnight_ian/bigc2elastic`.
3. Setup ENV variables by running `bigc2elastic --init`.
4. Run the utility with `bigc2elastic`.

Alternatively, if you want to pull the entire package down for development or
other specific use-cases:

1. First install nvm with `curl
   https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash`
   on Debian-based systems, update your environment settings with `source
   ~/.profile`, and install node with `nvm install latest` (alternatively,
   check out https://nodejs.org/en/download/ for other installation options).
2. Pull the repository down to your local machine by cloning the repository.
  - By [installing git](https://github.com/git-guides/install-git) and running:
    `cd ~ && git clone https://github.com/iangoodnight/bigc2elastic.git`
  - Or using cUrl with:
    `curl -L https://github.com/iangoodnight/bigc2elastic/archive/master.zip > \
    bigc2elastic.tar.gz && tar -zxvf bigc2elastic.tar.gz`
3. Install dependencies with `cd bigc2elastic/ && npm install` for the entire
   package, or with `cd bigc2elastic/ && npm install --production` to install
   the package without development dependencies.
4. Set your secret keys using `.env.example` and filling in the missing values
   and rename the resulting file from `.env.example` to `.env` with `mv
   .env.example .env`.
   - Or, run `./index --init` to set the ENV through the CLI.
5. If you have installed the entire package (with devDependencies) you can test
   your installation with `npm test`.
6. If everything looks right, you can run the application with `npm start`.


## Creating a cronjob

1. Setup your crontab with the command:
  `crontab -e`
2. Crontab uses the syntax `m h dom mon dow command` referring to `minute`,
   `hour`, `day of month`, `month`, and `day of week` respectively with `*`
   actiing as a wildcard for 'any'.  Occasionally, the relative paths called
   from node programs don't play nice with cronjobs, so we are going to start
   our <command> by changing directories to the root of bigc2elastic.  So, if we
   wanted to run our script once a week our crontab entry might look like:
   `0 5 * * 1 cd /home/<user>/bigc2elastic/ && ./index.js`

By default, the output from cronjobs goes to `/var/mail/`, but
you can override this by redirecting the output as part of the command. So the
crontab entry:
`0 5 * * 1 cd /home/<user>/bigc2elastic/ && ./index.js > ../run.log`
Runs the program and saves the output from that last job to the file
`/opt/bigc2elastic/run.log` for review.
