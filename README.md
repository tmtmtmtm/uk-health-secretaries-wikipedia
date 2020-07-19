Note: This repo is largely a snapshop record of bring Wikidata
information in line with Wikipedia, rather than code specifically
deisgned to be reused.

The code and queries etc here are unlikely to be updated as my process
evolves. Later repos will likely have progressively different approaches
and more elaborate tooling, as my habit is to try to improve at least
one part of the process each time around.

---------

Step 1: Check the Position Item
===============================

The Wikidata item for the
[Secretary of State for Health and Social Care](https://www.wikidata.org/wiki/Q3397406)
contains all the data expected already — nothing needs fixed up. It
would be nice if it tracked a little bit more of the history, as the
role has changed substantially over time, but we can add that later.

Step 2: Tracking page
=====================

PositionHolderHistory already exists; current version is
https://www.wikidata.org/w/index.php?title=Talk:Q3397406&oldid=1112210545
with 25 dated memberships and 2 undated; and 35 warnings.

Step 3: Set up the metadata
===========================

The first step in the repo is always to edit [add_P39.js script](add_P39.js) 
to configure the Item ID and source URL.

Step 4: Get local copy of Wikidata information
==============================================

    wd ee --dry add_P39.js | jq -r '.claims.P39.value' |
      xargs wd sparql office-holders.js | tee wikidata.json

Step 5: Scrape
==============

Comparison/source = [Secretary of State for Health and Social Care](https://en.wikipedia.org/wiki/Secretary_of_State_for_Health_and_Social_Care)

    wb ee --dry add_P39.js  | jq -r '.claims.P39.references.P4656' |
      xargs bundle exec ruby scraper.rb | tee wikipedia.csv

This required quite a bit of tweaking to get working. There are lots of
different tables, as the position has changed multiple times, and the
heading rows for them are quite awkward.

Step 6: Create missing P39s
===========================

    bundle exec ruby new-P39s.rb wikipedia.csv wikidata.json |
      wd ee --batch --summary "Add missing P39s, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

32 new additions as officeholders -> https://tools.wmflabs.org/editgroups/b/wikibase-cli/14bf447c479b2/

Step 7: Add missing qualifiers
==============================

    bundle exec ruby new-qualifiers.rb wikipedia.csv wikidata.json |
      wd aq --batch --summary "Add missing qualifiers, from $(wb ee --dry add_P39.js | jq -r '.claims.P39.references.P4656')"

12 additions made as https://tools.wmflabs.org/editgroups/b/wikibase-cli/21ad81e5fd153/

There are a few suggested corrections of start/end dates, but I'll wait
until everything has synced before looking at those.

Step 8: Refresh the Tracking Page
=================================

New version at https://www.wikidata.org/w/index.php?title=Talk:Q3397406&oldid=1234775419

Looks like we have a few issues to fix:

David Mellor and Brian Mawhinney are both listed as being the Secretary,
even though they were only Minister of State, so I've fixed their P39s
to that.

The overlap between John Reid and Patricia Hewitt can be resolved by
accepting the suggested change:

    wd uq 'Q332799$4CB1EC96-0623-4F41-977C-8095B3D057F5' P580 2005-05-05 2005-05-06

Similarly, the overlap between William Waldegrave and Virginia Bottomley
can be resolved by accepting

    wd uq 'Q332696$60FAFADD-76C9-45CC-9016-92776EF57BF9' P580 1992-04-09 1992-04-10

The start date for George Howard is also a little tricksy, as he became
the head of the first General Board of Health by virtue of already being
the Commissionser of Woods and Forests. I can't see exact start dates
for either of these, and I suspect we're going to have to add separate
positions for lots of these things anyway, as the history of the current
Health and Social Care overlaps with that of Work and Pensions between
the late 1960s and 1980s.

So I think for now we'll make do with the list at
https://www.wikidata.org/w/index.php?title=Talk:Q3397406&oldid=1234787081
