# yelp-system-design
Let's design a Yelp like service, where users can search for nearby places like restaurants, theaters, or shopping malls, etc., and can also add/view reviews of places.

Similar Services: Proximity server.

Proximity servers are used to discover nearby attractions like places, events, etc. If you haven’t used yelp.com before, please try it before proceeding (you can search for nearby restaurants, theaters, etc.) and spend some time understanding different options that the website offers. This will help you a lot in understanding this chapter better.

![](assets/hqdefault.jpg)

Have a look at the Requirements :
# Functional Requirement

Functional Requirements:

- Search all nearby places within a given radius.
- Add/delete/update Places.
- Add feedback/review about a place. The feedback may have pictures, text, and a rating.


# Non Functional Requirements
- Real-time search experience with minimum latency.
- Search should be scalable.

# Scalability Requirements:
- Yelp is search heavy system. A lot of search is happening every second compared to the insert/update.


# Basic Design and Database

## System APIs

Search Movies:

    search(api_dev_key, search_terms, user_location, radius_filter, maximum_results_to_return, 
    category_filter, sort, page_token)


## DB Design
Each Place can have the following fields:

    LocationID (8 bytes): Uniquely identifies a location.
    Name (256 bytes)
    Latitude (8 bytes)
    Longitude (8 bytes)
    Description (512 bytes)
    Category (1 byte): E.g., coffee shop, restaurant, theater, etc.

## NoSQL Tables

    PLACEDETAILS
    COMMENTS

SQL solution 

    Select * from Places where Latitude between X-D and X+D and Longitude between Y-D and Y+D

![](assets/dbdesign.png)

We have solved the problem for 1 person. lets discuss.

## Issues with SQL Solution:

Indexing on langitude and longitude is not efficient. Unnecessary table scan.

# Scalable Design

Let us understand the world better.

Geohash:

![](assets/geohash.jpg)

Lets zoom in further to the block 9

![](assets/world2.png)

And further more to the block 9v

![](assets/world3.png)



# Lets make another design

Now, in addition to storing the longitude and latitude of a place, a gridid will also be stored.

How to design a grid id for Yelp:

Design a hash function such that hash(long, lat) = gridid.

Now the search query looks like this:

    Select * from Places where Latitude between X-D and X+D and Longitude between Y-D and Y+D and GridID in (GridID, GridID1, GridID2, ..., GridID8)


Index on columng gridid will make this query fast.

Is this an optimal solution ? Can we optimized it further.

# Dynamic size grids 

Two grid of same area is not a similar grid for Yelp design.

Two grid with same number of point of interest is same kind of grid.

Consider the whole world as a single grid : 0
Hash function would be : hash(x,y) = 0
If there are more than 500(limit number) then divide this grid into 4 grids and redefine the hash function.

![](assets/quadtree.png)

Neighboring grid needs to be found.

Reverse Indexing:

    Grid 1 : loc1, loc2.....locn
    Grid 2 : loc1, loc2.....locn


The constraint will make sure the next transaction will fail and the second user will see error for seat reservation.

![](assets/pic2.png)


# Ranking
How about if we want to rank the search results not just by proximity but also by popularity or relevance?



# References

https://medium.com/@narengowda/bookmyshow-system-design-e268fefb56f5

https://www.youtube.com/watch?v=s4mO1Ak9G84

https://www.youtube.com/watch?v=lBAwJgoO3Ek&t=1379s

