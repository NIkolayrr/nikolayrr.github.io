---
title: Implementing Infinite Scroll with React Native FlatList and GraphQL - A Step-by-Step Guide
layout: post
---

Before we dive in, let's get a glimpse of what we're about to create.\
![Rick and Morty](https://media4.giphy.com/media/v1.Y2lkPTc5MGI3NjExY3dhYWhlZDU3aGc3Z3huaDl3aDFyczFub3NuNmJja2ltNGVhczg0NiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/cQdsmRdy9AGo0YST9m/giphy.gif)

# Step One - Setup the project and install dependencies

We will use Expo for this example. Let's run

```
npx create-expo-app RickAndMorty
cd RickAndMorty
```

Add all the required dependencies into package.json dependencies.

```
    "@apollo/client": "^3.8.3",
    "@react-navigation/native-stack": "^6.9.13",
    "expo": "~49.0.10",
    "expo-image": "~1.3.2",
    "expo-status-bar": "~1.6.0",
    "graphql": "^15.8.0",
    "lodash": "^4.17.21",
    "react": "18.2.0",
    "react-native": "0.72.4",
    "react-native-safe-area-context": "4.6.3",
    "react-native-screens": "~3.22.0"
```

then run

```
npm install
```

# Step Two - Setup ApolloClient

We'll utilize the Rick and Morty API [https://rickandmortyapi.com/graphql](https://rickandmortyapi.com/graphql). Our initial step is to set up our ApolloClient.

```
// queries.js
import { ApolloClient, InMemoryCache } from '@apollo/client'

export const client = new ApolloClient({
  uri: 'https://rickandmortyapi.com/graphql/',
  cache: new InMemoryCache({}),
})
```

Now let's wrap our app with ApolloProvider enabling you to access it from anywhere in your component tree.

```
// App.js
import { NavigationProvider } from './Navigation'
import { NavigationContainer } from '@react-navigation/native'
import { ApolloProvider } from '@apollo/client'
import { client } from './queries'

export default function App() {
  return (
    <NavigationContainer>
      <ApolloProvider client={client}>
        <NavigationProvider />
      </ApolloProvider>
    </NavigationContainer>
  )
}

```

```
// NavigationProvider.js

import { createNativeStackNavigator } from '@react-navigation/native-stack'
import { HomeScreen } from './screens/HomeScreen'

const Stack = createNativeStackNavigator()

export const NavigationProvider = () => {
  return (
    <Stack.Navigator>
      <Stack.Screen name='Home' component={HomeScreen} />
    </Stack.Navigator>
  )
}

```

Let's also create our Home Screen which will be located under a screen folder

```
// HomeScreen.js

import { View, Text, StyleSheet } from 'react-native'

import { SafeAreaView } from 'react-native-safe-area-context'

export const HomeScreen = () => {
  return (
    <SafeAreaView style={styles.safeArea}>
      <View style={styles.container}>
        <Text>Home</Text>
      </View>
    </SafeAreaView>
  )
}

const styles = StyleSheet.create({
  safeArea: {
    flex: 1,
    backgroundColor: 'white',
  },
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
})

```

We can utilize the useQuery hook from Apollo to fetch character data from the API and create a simple list displaying the names of the first 20 characters.

```
import { gql, ApolloClient, InMemoryCache } from '@apollo/client'

export const GET_CHARACTHERS = gql`
  query GetCharacters($page: Int!) {
    characters(page: $page) {
      results {
        id
        name
        status
        species
        image
        location {
          id
          name
        }
      }
    }
  }
`

export const client = new ApolloClient({
  uri: 'https://rickandmortyapi.com/graphql/',
  cache: new InMemoryCache({}),
})

```

We can use useQuery hook from apollo to retrive the character data from the API and setup a plain list containing the names of the first 20 characters.

```
// HomeScreen.js
import { useQuery } from '@apollo/client'
import { View, Text, StyleSheet, FlatList } from 'react-native'

import { SafeAreaView } from 'react-native-safe-area-context'
import { GET_CHARACTHERS } from '../queries'
import _ from 'lodash'

const handleRenderItem = ({ item: charcter }) => {
  return (
    <View>
      <Text>{charcter.name}</Text>
    </View>
  )
}

export const HomeScreen = () => {
  const { loading, error, data } = useQuery(GET_CHARACTHERS, {
    variables: { page: 0 },
  })

  if (loading) {
    return (
      <View>
        <Text>Loading!</Text>
      </View>
    )
  }

  if (error) {
    return (
      <View>
        <Text>Error</Text>
      </View>
    )
  }

  return (
    <SafeAreaView style={styles.safeArea}>
      <View style={styles.container}>
        {!_.isEmpty(data) && (
          <FlatList style={styles.listStyles} data={data?.characters?.results} renderItem={handleRenderItem} />
        )}
      </View>
    </SafeAreaView>
  )
}
```

Next, we'll enhance the handleRenderItem function by incorporating some styling and providing additional character information.

```
// HomeScreen.js
import { Image } from 'expo-image'

...
const handleRenderItem = ({ item: charcter }) => {
  return (
    <View key={charcter.id} style={styles.item}>
      <Image style={styles.image} source={charcter.image} contentFit='cover' />
      <View style={styles.itemDetails}>
        <Text style={styles.name}>{charcter.name}</Text>
        <Text style={styles.details}>Species: {charcter.species}</Text>
        <Text style={styles.details}>Status: {charcter.status}</Text>
        <Text style={styles.details}>Last Known Location: {charcter.location.name}</Text>
      </View>
    </View>
  )
}

...

const styles = StyleSheet.create({
  safeArea: {
    flex: 1,
    backgroundColor: 'white',
  },
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  image: {
    flex: 1,
    height: 150,
  },
  name: {
    fontSize: 22,
    marginVertical: 10,
  },
  textInput: {
    width: '95%',
    height: 40,
    borderWidth: 1,
    padding: 5,
    marginBottom: 10,
  },
  listStyles: {
    width: '100%',
  },
  item: {
    borderWidth: 1,
    marginVertical: 10,
    marginHorizontal: 10,
    flexDirection: 'row',
  },
  itemDetails: {
    flex: 1,
    marginLeft: 10,
  },
  details: {
    marginVertical: 2,
  },
})

```

We've created a list and fetched data from the API, but now we need to set up pagination. First, let's add the **onEndReached** handler to the FlatList to detect when to make new requests. Then, we can use **fetchMore** from useQueries to update the page variable and make additional requests.

```
// HomeScreen.js

  const { loading, error, data, fetchMore } = useQuery(GET_CHARACTHERS, {
    variables: { page: 0 },
  })
  const PAGE_ITEMS = 20

	...

  const hanndleEndReach = () => {
    fetchMore({
      variables: { page: data.characters.results.length / PAGE_ITEMS + 1 },
      updateQuery: (previousQueryResult, { fetchMoreResult }) => {
        return {
          characters: {
            results: [...previousQueryResult.characters.results, ...fetchMoreResult.characters.results],
          },
        }
      },
    })
  }

  return (
    <SafeAreaView style={styles.safeArea}>
      <View style={styles.container}>
        {!_.isEmpty(data) && (
          <FlatList
            style={styles.listStyles}
            data={data?.characters?.results}
            renderItem={handleRenderItem}
            onEndReached={hanndleEndReach}
          />
        )}
      </View>
    </SafeAreaView>
  )
}

```

We might receive a warning related to **ensuring that all objects of type Characters have an ID or a custom merge function.** To address this issue, let's include typePolicies in the InMemoryCache.

```
// queries.js

export const client = new ApolloClient({
  uri: 'https://rickandmortyapi.com/graphql/',
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          characters: {
            keyArgs: false,
            merge: true,
          },
        },
      },
    },
  }),
})
```

The final step is to incorporate a loading indicator when we fetch additional characters. To achieve this, we'll include **networkStatus** from **useQueries** and set up a listener for **notifyOnNetworkStatusChange**. We can also remove the loading check.

```
// HomeScreen.js
import { NetworkStatus, useQuery } from '@apollo/client'
import { View, Text, StyleSheet, FlatList, ActivityIndicator } from 'react-native'
import { Image } from 'expo-image'

import { SafeAreaView } from 'react-native-safe-area-context'
import { GET_CHARACTHERS } from '../queries'
import _ from 'lodash'

const handleRenderItem = ({ item: charcter }) => {
  return (
    <View key={charcter.id} style={styles.item}>
      <Image style={styles.image} source={charcter.image} contentFit='cover' />
      <View style={styles.itemDetails}>
        <Text style={styles.name}>{charcter.name}</Text>
        <Text style={styles.details}>Species: {charcter.species}</Text>
        <Text style={styles.details}>Status: {charcter.status}</Text>
        <Text style={styles.details}>Last Known Location: {charcter.location.name}</Text>
      </View>
    </View>
  )
}

export const HomeScreen = () => {
  const { loading, error, data, fetchMore, networkStatus } = useQuery(GET_CHARACTHERS, {
    variables: { page: 0 },
    notifyOnNetworkStatusChange: true,
  })
  const PAGE_ITEMS = 20

  if (error) {
    return (
      <View>
        <Text>Error</Text>
      </View>
    )
  }

  const hanndleEndReach = () => {
    fetchMore({
      variables: { page: data.characters.results.length / PAGE_ITEMS + 1 },
      updateQuery: (previousQueryResult, { fetchMoreResult }) => {
        return {
          characters: {
            results: [...previousQueryResult.characters.results, ...fetchMoreResult.characters.results],
          },
        }
      },
    })
  }

  return (
    <SafeAreaView style={styles.safeArea}>
      <View style={styles.container}>
        {!_.isEmpty(data) && (
          <FlatList
            style={styles.listStyles}
            data={data?.characters?.results}
            renderItem={handleRenderItem}
            onEndReached={hanndleEndReach}
          />
        )}
        <View style={{ marginVertical: 10 }}>
          {networkStatus === NetworkStatus.fetchMore && (
            <View style={{ flexDirection: 'row' }}>
              <Text style={{ marginRight: 10 }}>Fetching Data</Text>
              <ActivityIndicator />
            </View>
          )}
        </View>
      </View>
    </SafeAreaView>
  )
}
```

And there you have it! 🎉 \
We now have a fully functioning Infinite Scroll. Thank you for your attention!
