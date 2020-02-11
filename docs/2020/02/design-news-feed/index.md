# Design News Feed System


http://blog.gainlo.co/index.php/2016/03/29/design-news-feed-system-part-1-system-design-interview-questions/

## Requirements

### Features
1. CRUD posts
2. Comment on posts
3. Share posts

### What is in a news feed post?
1. Author
2. Content
3. Media (photos, videos, .etc)
4. Comments and replies
5. Operations:
   1. CRUD
   2. Comment and reply on posts
   3. Share posts

### What is on news feed page?
1. Sequence of posts
2. Query or filter posts
3. Operations:
   1. Load more posts
   2. Delete posts I don't want to see

## Estimations (scale requirements)

1. How many users?
2. How many new feed generated one day?
3. How many could a user be followed by another?

## Design Goals

TBD

## Skeleton of the Design

1. Based on the requirements, we need some schema to store user and feed object, that is the **data model**. And designing the data model should take CRUD, comment and reply operations of feeds into consideration. How to trade off when we try to optimize the system on read/write?

2. To display feeds in sequence, do we need some more that just **order** by time?

3. Do we need to consider the scalability? Our system could be easy under hundreds of users, but can it be efficient for millions or even billions of users? Our system should be **scalable**.

### Data model

### Feed ranking

### Feed publishing

## Deep Dive

### Data model

#### If using relational database
There are two basic object: **user** and **feed**.
- For user object, we can store UserID, Name, RegistrationDate, .etc.
- For feed object, we can store FeedID, FeedType, Content, Media (image/video), .etc.

There are also some relations: **user_friend** - the friends of a user, **user_feed** - the feeds published by a user.
- user_friend is a many to many relationship. We could use a table with two UserID, one for the user, the other for his/her friend.
- user_feed is one to many relationship. To reduce the join number, we could add column UserID to feed table. This approach is called **denormalization**, compared with **normalization** which would add a new table user_feed with two columns, one for UserID, the other for FeedID.

For example, we need to query all the feeds published by someone's friends. We need to:
1. Join user and user_friend tables to get all UserID of his/her friends.
2. Get all the feeds from feed table with query of friends' UserIDs.

#### If using non-relational database, like nosql.
**TBD**

### Ranking

The most straightforward way to rank feeds is **by the time** it was created. But that is not enough. "**Important**" feeds are ranked on top.

The reason to have better ranking algorithm is that better ranking algorithm improves the users stickiness, users retention, ads revenue etc..

A common strategy is to calculate a feed score based on various features and rank feeds by its score.

We can select several features that are mostly relevant to the importance of the feed, e.g. share/like/comments numbers, time of the update, whether the feed has images/videos etc.. And then, a score can be computed by these features, maybe a linear combination.

#### EdgeRank
A famous feed ranking algorithm used by Facebook.

For each news update you have, whenever another user interacts with that feed, they're creating what Facebook calls an **Edge**, which includes actions like `like` and `comment`.

The features used to evaluate the importance of an update/feed are **affinity score**, **edge weight** and **time decay**.

- Affinity score.

    It evaluates how close you are with the user. For instance, you are more likely to care about feeds from your close friends instead of someone you just met once.

    Various factors can be used to reflect how close two people are.

    First of all, **explicit interactions** like comment, like, share etc. are strong signals we should use. Apparently, each type of interaction should have **different weight**. For instance, comments should be worth much more than likes.

    Secondly, we should also track the **time factor**. Perhaps you used to interact with a friend quite a lot, but less frequent recently.

- Edge weight.

    It reflects importance of each edge. For instance, comments are worth more than likes.

- Time decay.

    The older the story, the less likely users find it interesting.

How to rank feeds by these three features?
1. For each feed you create
2. Multiply these factors for each Edge
3. Then add the Edge scores up
4. You have an update's EdgeRank. And the higher that is, the more likely your update is to appear in the user's feed.


### Feed publishing

When a user loads all the feeds from his/her friends, it can be an extremely costly action. Remember that a user can have thousands of friends and each of them can publish a huge amount of updates especially for high profile users.

How to optimize and scale the feed publishing system?

There are two common approaches: push and pull.

#### push

Once a user has published a feed, we immediately pushing this feed to all his friends.

- Pros, when fetching feed, you don't need to go through your friends list and get feeds for each of them. It significantly reduces read operation.
- Cons, it increases write operation especially for people with a large number of friends.

#### pull

Feeds are only fetched when users are loading their home pages. So feed data doesn't need to be sent right after it's created.

- Pros, it optimizes for write operation.
- Cons, can be quite slow to fetch data.

The process of pushing an activity to all your friends or followers is called a fanout. The push approach is also called fanout on write, while the pull approach is fanout on load.

#### Seleteive fanout - combination of push and pull

If you are mainly using push model, what you can do is to disable fanout for high profile users and other people can only load their updates during read.

Also, once a user publish a feed, we can also limit the fanout to only his/her active friends.
