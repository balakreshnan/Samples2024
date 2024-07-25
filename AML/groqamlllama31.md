# Groq LLama 3.1 8B/80B- Azure Machine Learning

## Introduction

- Using Groq building Gen AI application
- Using Azure Machine Learning
- Using LLama 3.1 using Groq inferencing

## Pre-requisites

- Azure Subscription
- Azure Machine Learning
- Groq cloud developer access
- Get a API key from Groq

## Steps

- Install groq sdk

```
%pip install groq
```

- Import the libraries

```
from dotenv import dotenv_values

config = dotenv_values("env.env")
```

- Set the api key, from the config

```
grow_key = config["GROQ_API_KEY"]
```

- now call the llama through groq inferencing

```
from groq import Groq

client = Groq(api_key=grow_key)
completion = client.chat.completions.create(
    model="llama-3.1-8b-instant",
    messages=[
        {
            "role": "user",
            "content": "Show me places to visit in NY?"
        }
    ],
    temperature=1,
    max_tokens=1024,
    top_p=1,
    stream=True,
    stop=None,
)

for chunk in completion:
    print(chunk.choices[0].delta.content or "", end="")
```

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/groqllama31-1.jpg 'RagChat')

- Analyze the output

```
The Empire State! New York has endless options for entertainment, culture, and outdoor activities. Here are some top places to visit in NY:

**New York City:**

1. **Statue of Liberty** and **Ellis Island**: Take a ferry to Liberty Island to see the iconic statue up close and visit the Ellis Island Immigration Museum.
2. **Central Park**: A massive urban park in the heart of Manhattan, offering walking paths, lakes, and plenty of green space.
3. **Metropolitan Museum of Art**: One of the world's largest and most famous museums, with a collection that spans over 5,000 years of art history.
4. **Times Square**: The bustling heart of New York City, known for its bright lights, giant billboards, and live performances.
5. **9/11 Memorial & Museum**: A poignant tribute to the victims of the 9/11 attacks, featuring two massive waterfalls and a museum with artifacts and stories.
6. **Rockefeller Center**: A complex of buildings in Midtown Manhattan that includes NBC studios, a skating rink, and a Christmas tree lighting ceremony.
7. **Broadway**: Catch a musical or play on one of the world's most famous theater scenes.
8. **High Line**: An elevated park built on an old rail line, offering great views of the Hudson River and the city.
9. **Brooklyn Bridge**: Walk across this iconic bridge for stunning views of the Manhattan skyline and the East River.
10. **Madame Tussauds**: A wax museum featuring life-like replicas of famous people, from celebrities to historical figures.

**Upstate New York:**

1. **Niagara Falls**: A breathtaking waterfall on the border of New York and Ontario, Canada.
2. **Adirondack Park**: A vast wilderness area with mountains, lakes, and forests, perfect for outdoor adventures like hiking, skiing, and fishing.
3. **Saratoga Springs**: A charming town known for its horse racing, spas, and natural springs.
4. **Letchworth State Park**: A beautiful park with waterfalls, hiking trails, and a scenic gorge.
5. **Albany**: The state capital, featuring historic sites like the New York State Capitol building and the Empire State Plaza.

**The Hudson Valley:**

1. **Walkway Over the Hudson**: A scenic pedestrian bridge offering stunning views of the Hudson River.
2. **Dutchess County**: A picturesque region with rolling hills, apple orchards, and historic towns like Beacon and Cold Spring.
3. **Rhinebeck**: A quaint village with a rich history, shopping, and dining.
4. **Hudson River Valley Greenway**: A scenic drive or bike ride along the river, passing through charming towns and villages.
5. **Sleepy Hollow**: The legendary home of Headless Horseman and Washington Irving's "The Legend of Sleepy Hollow."

**Long Island:**

1. **Jones Beach State Park**: A popular beach with music venues, water sports, and a historic pier.
2. **The Hamptons**: An affluent coastal region known for its beaches, wineries, and celebrity homes.
3. **Montauk**: A scenic lighthouse at the eastern tip of Long Island, offering stunning ocean views and hiking trails.
4. **Long Island Museum**: A unique museum showcasing the region's history, art, and culture.
5. **CIA (Culinary Institute of America) Hyde Park**: A world-renowned culinary school with a focus on American cuisine and a B&B-style campus.

**The Finger Lakes Region:**

1. **Corning Museum of Glass**: A world-class museum showcasing the art of glassmaking and a collection of over 45,000 objects.
2. **Watkins Glen State Park**: A beautiful park with waterfalls, hiking trails, and scenic gorges.
3. **Taughannock Falls State Park**: A park featuring a stunning 215-foot waterfall and scenic hiking trails.
4. **Seneca Lake**: A picturesque lake with scenic boat cruises and wine tastings.
5. **Ithaca**: A charming college town home to Cornell University and Ithaca College, with a vibrant arts and food scene.

These are just a few of the many amazing places to visit in New York. Depending on your interests and the time of year you visit, there are plenty of other attractions and activities to explore.
```

- now change the model 3.1 70B

![info](https://github.com/balakreshnan/Samples2024/blob/main/AML/images/groqllama31-2.jpg 'RagChat')

```
The Empire State! New York has endless options for visitors, from iconic landmarks to world-class museums, beautiful parks, and vibrant neighborhoods. Here are some top places to visit in New York:

**Iconic Landmarks:**

1. **Statue of Liberty and Ellis Island**: Take a ferry to Liberty Island to see the iconic statue up close and visit the Ellis Island Immigration Museum.
2. **Central Park**: A peaceful oasis in the middle of Manhattan, offering walking paths, lakes, and plenty of green space.
3. **Empire State Building**: Enjoy panoramic views of the city from the observation deck of this iconic skyscraper.
4. **Brooklyn Bridge**: Walk across this historic bridge for spectacular views of the Manhattan skyline and the East River.
5. **Times Square**: The heart of New York's entertainment district, known for its bright lights, giant billboards, and lively atmosphere.

**Museums:**

1. **The Metropolitan Museum of Art**: One of the world's largest and most famous museums, with a vast collection of art and artifacts from around the world.
2. **American Museum of Natural History**: Explore the wonders of the natural world at this iconic museum, home to over 30 million specimens.
3. **The Museum of Modern Art (MoMA)**: A world-renowned institution dedicated to modern and contemporary art.
4. **9/11 Memorial & Museum**: A poignant tribute to the victims of the 9/11 attacks, featuring two massive reflecting pools and a museum with artifacts and stories.
5. **Guggenheim Museum**: Designed by Frank Lloyd Wright, this iconic museum is home to an impressive collection of modern and contemporary art.

**Neighborhoods:**

1. **Greenwich Village**: Known for its charming streets, historic brownstones, and trendy restaurants and bars.
2. **SoHo**: Short for "South of Houston," this neighborhood is famous for its cast-iron buildings, upscale shopping, and art galleries.
3. **Chinatown and Little Italy**: Experience the vibrant culture and cuisine of these two neighboring districts.
4. **Williamsburg**: A hip neighborhood in Brooklyn known for its indie music scene, trendy bars, and vintage shops.
5. **Harlem**: A historic neighborhood with a rich cultural heritage, home to the Apollo Theater, jazz clubs, and soul food restaurants.

**Other Attractions:**

1. **Rockefeller Center**: A complex of buildings in Midtown Manhattan that includes NBC studios, a skating rink, and the famous Christmas tree lighting ceremony.
2. **One World Trade Center**: The tallest building in the Western Hemisphere, with an observation deck offering stunning views of the city.
3. **Coney Island**: A classic Brooklyn beachside neighborhood with an amusement park, boardwalk, and delicious seafood.
4. **New York Public Library**: A beautiful Beaux-Arts building with a stunning reading room and exhibitions.
5. **Broadway**: Catch a performance of a hit musical or play on the Great White Way.

**Day Trips:**

1. **Niagara Falls**: Take a bus or train to the famous waterfalls, located just a few hours north of the city.
2. **The Hamptons**: A popular summer destination on Long Island, known for its beautiful beaches, wineries, and celebrity homes.
3. **The Bronx**: Visit the New York Yankees' stadium, the Bronx Zoo, and the beautiful Van Cortlandt Park.
4. **Staten Island**: Take a ferry to the least-visited borough, with its beautiful beaches, historic sites, and stunning views of the Verrazano-Narrows Bridge.
5. **The Hudson Valley**: Explore the scenic countryside north of the city, with its picturesque towns, wineries, and historic sites.

These are just a few of the many amazing places to visit in New York. Whether you're interested in history, art, food, or entertainment, there's something for everyone in the city that never sleeps!
```

- Change the questions and see the output