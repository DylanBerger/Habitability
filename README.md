# Overview 

In a previous project, I created a classifier which well, classifies planets. I personally find potentially habitable planets to be so interesting so I was like, lets find some! My interpretation of potential habitability was based on the earth similarity index (similarity to earth in terms of flux and size), and whether it lies in the habitable zone of the star (I will explain my process for that later).

# Data Preperation + Earth Similarity Index 

I first started by taking all the Super Earth and Terrestrial planets out of the full dataset, and making a new one. I did this because those two categories are the only ones that could support life as we know it. 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/fd344df1-1761-4a5a-a6ea-32205bb85b66)

The next thing I did was prepare to calculate the earth similarity index for each planet. I need two variables:
- Flux
- Planet Radius

Planet Radius is easily accessible, but there are a lot of null values with regards to the flux, so we need to do some calculations. 

This is the formula for flux: 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/4cc4bc03-88e9-41a2-aa4f-49245107f6a0)

Rplanet is the orbital radius, and Pstar/Psun is the luminosity relative to our sun. We have orbital radius (semi major axis) in our data, but we need to calculate luminosity.

Here is the formula for luminosity (T being stellar temperature and R as stellar radius): 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/a896cd34-637d-4730-baef-e590555fd08f)

Which we (thank god) have everything we need to calculate that. 

I then wrote a function to calculate luminosity, and then calculated it for every planet in my dataset using data.apply , creating a new column as result called "st_luminosity"

![image](https://github.com/DylanBerger/Habitability/assets/82914031/d5f59b6e-87ba-401f-a6e2-6c6232b9922c)

Now that we have luminosity, we can calculate stellar flux. 

I wrote a function to calculate flux, and then applied that to every row which has a null value in my flux column (pl_insol). 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/dfed0baf-37e0-4efa-8a0a-6a24f560cab0)

Now that we finally have stellar flux, and exoplanet radius, we can calculate the exoplanet earth similarity index, which has the following formula: 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/5a61d490-bd49-4b31-821d-c3aa014585da)

Where S is stellar flux, R is radius, S⊕ is Earth's solar flux, and R⊕ is Earth's radius (formula from the The Planetary Habitability Laboratory). 

I proceeded to (you guessed it) write a function to calculate ESI for every row in my dataset

![image](https://github.com/DylanBerger/Habitability/assets/82914031/3264b4c2-6a30-464c-9806-094ea3f8b377)

![image](https://github.com/DylanBerger/Habitability/assets/82914031/18d51e1f-994d-4c06-bceb-e0ce513c3436)

After that, I decided that I was too lazy to drop every column manually so I just created a new dataframe with only the columns I needed for the next stage, which is exploring the Earth Similarity Index!

- Also for context, a ESI of 0.95 would indicate that the planet is 95% similar to earth. 

# ESI Exploration 

I first started with a heat map and contour map (colors indicating closeness to 1.0 on ESI scale)

![image](https://github.com/DylanBerger/Habitability/assets/82914031/8d505d2d-9765-4e1b-83dd-7c1a3d1e729a)

![image](https://github.com/DylanBerger/Habitability/assets/82914031/ac1fb174-c957-450b-bbfa-3e8f99129e65)

Above: A flux of 1 and radius of 1, is earth. The graphs show how close each planet is to earth.

This showed that very few Super-Earths and Terrestrial planets actually are similar to earth despite them likely being rocky and around earth size. As you can see below, only 3 or so percent had an ESI above 0.7 which is when a planet is considered to be "earth-like", it isn't a meausre of habitability by any means, but it is a start. 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/0c7590bb-b6f4-4c2d-8f43-2cd5e44549a9)

I then took only the planets with a ESI above 0.7 and saved them as new dataset.

![image](https://github.com/DylanBerger/Habitability/assets/82914031/44ee1156-ce84-403f-9ed0-e98572898e4e)

As we can see, these are overwhelmingly Super-Earth planets. 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/8e3c8610-80b2-4ae1-a6b0-ef0d25b0c142)

This is obviously attributed to the massive class imbalance present from the start.

Next step is to plot equilibrium temperature and radius. Temperature is the new variable we are introducing (and is perhaps one of the most important for potential habitability). 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/4f8e11f1-3c99-4d3c-b7a1-6e3639dfd291)

Here is flux and radius as well. 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/4be85b4b-1e8c-47da-9209-ee910d5aa64e)

Figures Above: The two black lines relate to the habitable zone (will explain later in this) and the red line is the earths equilibrium temperature.

What these graphs tell us is that despite ESI not being a measure of habitability, there may be a correlation. As the planets with equilibrium temperatures and flux similar to Earths (earths being the red line) happen to be higher up on the ESI scale. 

Correlation is not causation, but the ESi certainly can help us narrow down which planets are more likely to be habitable. 

# Habitable Zone 

The next step is to sort the planets by whether they are in the habitable zone or not. 

I wrote an algorithm to approximate it (habitable_zone.ipynb) based on the process outlined by https://www.planetarybiology.com/calculating_habitable_zone.html. What I found however, was that when I sorted the planets, very few actually were in the calculated habitable zone. I was trying to get a list similar to the one on https://phl.upr.edu/projects/habitable-exoplanets-catalog, but couldn't with that process. Either the calculation is a super conservative estimation, or I needed a different way to find whether a planet is in the habitable zone.

My solution was to approximate the boundaries based on equilibrium temperature, which is the range through which a planet can habor water, the driving factor behind habitable zone calculation to begin with. This temperature range is from 175K to 275K as outlined by a paper by Berkeley Scientists https://ulab.berkeley.edu/static/doc/posters/s181.pdf

Sorting the planets like this yielded a result much similar to the one found on the The Planetary Habitability Laboratory Website. 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/4d89e255-921f-4d8c-9d10-5d1f4c303a9a)

I then did similar heat map:

![image](https://github.com/DylanBerger/Habitability/assets/82914031/0ab0382c-8b95-4fe4-940a-b05f1375a990)

Now normally, this would be the end, but I wanted to see if there are any potential superhabitable planets out of my list. 

The conditions are as follows (found on https://handwiki.org/wiki/Astronomy:Superhabitable_planet#Profile_summary): 

- Mass around 2 Earths
- Orbiting a K-Type Star (these stars last longer, emit less dangerous radiation, are less volatile)
- Radius around 1.2 or 1.3 Earths
- Star Age older than 4.5 billion years but younger than 7 billion years
- Surface Temperature up to 10 Kelvin higher than Earths

I first started sorting by size (x-axis is radius, and y-axis is mass):

![image](https://github.com/DylanBerger/Habitability/assets/82914031/4400ae74-1af1-4b4f-9ba8-ed407d21dda9)

Which yields four that meet the conditions.

Next was by star type: 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/d6830ef3-15e7-410c-bacf-29a791fd8fe7)

Only Kepler 442 b met the conditions. 

This happens to be the only exoplanet that the website identified as a potential superhabitable planet.

Next I wanted to analyze how well it did with the other conditions: 

![image](https://github.com/DylanBerger/Habitability/assets/82914031/a7cb354b-3ff5-4171-8ee1-2a41e762f9fe)

Star age is too young and the temperature is a bit cool, but this by no means eliminates it.

![image](https://github.com/DylanBerger/Habitability/assets/82914031/85a8e5a5-6cfe-4a2a-905c-27a1e8262076)

Additionally, the earth similarity index is high and the flux is lower than earths so presumably it would be bombarded with less radiation. 

Kepler 442 b seems to be the best candidate so far.

The 23 other superhabitable planet candidates are either unconfirmed planets (most of them) or do not orbit K-type stars.

Anyways this project was fun, see you next time!
