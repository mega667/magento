---
layout: m1-documentation
title: Indexing Queue
permalink: /doc/m1/indexing-queue/
description: Learn how the indexing queue works in Algolia extension for Magento
---

<div class="alert alert-warning">
If you are having any issues with your data, indexes, or queue, please check our <a href="/magento/doc/faq-support-data">troubleshooting guide</a>;
</div>

The **index queue** manages all uploads to Algolia. Before your data can be searched, it must be uploaded to Algolia. This process is called indexing, which the extension does automatically - via the queue.

<div class="alert alert-warning">
Also, if you want to know more about indexing in general, please check out our <a href="/magento/doc/m1/indexing">Indexing overview</a>;
</div>

The extension uses the **queue** to keep your data (products, categories, pages, etc.) synchronized with Algolia in real time. Note that by real-time, we are referring to the mechanism by which every update to your Magento data will trigger an event in the extension which will in turn initiate a reindex. This provides the best, most up-to-date search experience for your customers.

Queue operations run asynchronously, allowing the administrator to continue working while the queue refreshes Algolia in the background.

## The Indexing queue

To enable the indexing queue navigate to **System > Configuration > Algolia Search > Indexing Queue / Cron tab** in your Magento administration.

All queued operations will appear in a database table called `algoliasearch_queue`.

Additionally, you can set how many jobs will be processed at a time. The default number of jobs is 10. You can adjust this number to fit your catalog and the server your Magento store runs on.

Once the queue is enabled, you need to set up the process that will run it. There are several ways to do this.

## With cron

The standard set up is to process queued jobs at a regular time interval. To do this, configure the following crontab entry:

```sh
*/5 * * * * php -f /absolute/path/to/magento/shell/indexer.php -- -reindex algolia_queue_runner
```

This will run N jobs every 5 minutes depending on your queue configuration.

## Without cron

You can also process the queue manually from the command line:

```sh
php -f /absolute/path/to/magento/shell/indexer.php -- -reindex algolia_queue_runner
```

This will execute a single instance of a cron job. Note that this will not always empty the queue, as there might be more jobs than one instance can handle.

## Full section reindexing

### With the indexing queue enabled

When the indexing queue is enabled, products are reindexed by using temporary indices. Instead of being sent to the production index (which could cause temporary duplication or inconsistency), the products are uploaded to a temporary index. Then, only when all the products are pushed, the temporary index is changed to become the production index. This approach has several advantages:

1. Higher re-indexing speed
2. Avoid potential inaccuracies with deleted products
3. Lower number of operations needed

All changes done by re-indexing will be visible in search results once the whole process has completed and the production indices have been replaced.

### With the indexing queue disabled

When the indexing queue is disabled, the full product re-index has to process the whole catalog synchronously. Updates must be pushed to Algolia as well as deletes to remove inactive products.

This takes more time and more resources. It is also a little bit less reliable as some deleted products may not be processed and removed from Algolia’s indices.

<div class="alert alert-warning">
    <i class="fa fa-exclamation-triangle"></i>
    Enabling the indexing queue is highly recommended for doing any full reindexing on large catalogs.
</div>

<div class="alert alert-danger">
	<i class="fa fa-exclamation-triangle"></i>
	When the indexing queue is disabled, every indexing job <i>(complete re-indexing, update/deletion/update of products or categories, etc.)</i> will run synchronously. Trying to synchronously index too many objects at a time may trigger PHP timeouts. See more about <a href="/magento/doc/faq-support-data/#my-data-is-too-large">large indexes</a>.
</div>

## Automatic indexing

By default, the extension indexes each change or deletion of product or category and this change is propagated to Algolia immediately. It's useful as it keeps the data in Algolia in sync with what is in your Magento. But if you want to prevent this behavior, you can do it by changing the indexer's mode to "Manual Update".

This change will prevent the indexer to index every single change of a product or a category immediately.

When you switch the mode to "Manual Indexing", you'll need to run full product and category reindex on a regular basis - for example every night - to keep your data synchronized with Algolia.

## Manual reindexing

If you want to completely reindex your catalog, you can do it in two ways:

#### Via administration interface

In your Magento administration navigate to **System > Index Management** and there hit **Reindex** button right next to the indexers you want to reindex:

<figure>
    <img src="../../../img/indexers.png" class="img-responsive" alt="Indexers">
    <figcaption>Algolia indexers</figcaption>
</figure>

#### Via command line

If you want to completely reindex your catalog, you can use the command line.

For example, here is the command for a complete product reindex:

```sh
php -f /absolute/path/to/magento/shell/indexer.php -- -reindex algolia_search_indexer
```

You can use more than one indexer name in the command.

Names of other Algolia indexers:

- `algolia_search_indexer` - reindexes all products
- `algolia_search_indexer_cat` - reindex all categories
- `algolia_search_indexer_pages` - reindexes all CMS pages
- `search_indexer_suggest` - reindexes all search query suggestions
- `search_indexer_addsections` - reindexes all data from additional sections
- `algolia_queue_runner` - process jobs in indexing queue

## Emptying the queue

If you want to clear the queue in one instance, you will need to bypass the cron and process the entire queue in one pass. You do this with by setting the variable EMPTY_QUEUE=1:

```sh
EMPTY_QUEUE=1 php path/to/magento/bin/magento indexer:reindex algolia_queue_runner
```

If you want to process the entire queue in one pass you can run:

```sh
EMPTY_QUEUE=1 php -f /absolute/path/to/magento/shell/indexer.php -- -reindex algolia_queue_runner
```

Running this will attempt to empty the queue. However, it might not always be successful. Errors can occur - network timeouts, or more often, your data is too large for one job. In that case, the job will fail, the queue will not be empty, and therefore some of your data will not have been sent to Algolia. Please go here how to resolve [indexing errors](/magento/doc/faq-support-data/#common-errors).
