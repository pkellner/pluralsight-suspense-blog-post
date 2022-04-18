# React 18 and Suspense

[React 18 has shipped](https://reactjs.org/blog/2022/03/29/react-v18.html) as of March 29th, 2022. The new Concurrent Mode feature [Suspense](https://reactjs.org/docs/react-api.html#reactsuspense) is 100% production ready. However, unless you have the development resources necessary to use [GraphQL](https://graphql.org/) with [Relay](https://relay.dev/), you likely will not be able to use Suspense in your React Apps. Suspense will be coming for everyone and if you’re a React developer and want to be ready there is much you can understand and prepare for. In this post, you'll learn what you will eventually need to **integrate Suspense into your React app**.

## What is Suspense and Do I Need It?

Suspense is the first feature released by the React that takes advantage of the new rendering engine built into React 18. That engine is referred to as the one using **Concurrent Mode**. Benefits of using it include apps having better **more responsive UIs** while using less browser resources, as well as giving developers and designers a **more intuitive API to work with**.

Suspense has been in the making by the Facebook React team for over 3 years. It fundamentally changes how React determines what to render on the web page based on your React app’s component state changes. That is, if there is any of your React app’s data (also known as state data) that changes, and that does not require a full page re-render from the server, the React rendering engine does that update.

The main difference between [Concurrent Mode](https://17.reactjs.org/docs/concurrent-mode-intro.html), and the React rendering engine prior, is what happens when a web page updates a list based on some interruption, like typing in a text box that is used as a filter items on a list. Without Concurrent Mode, it's possible that all the items in the list will get updated by React causing both the page to feel sluggish to the user, as well as the browser using lots of your computer resources to make the app work at all. If your browser allows you to view the animated gif below, this is what you might see.

![](not-responsive-keyboard-demo-sm.gif)

If, on the same computer, you were using an app that implemented Concurrent Mode, meaning that as the user typed into the search box, only what was necessary in the DOM was updated, you might see a UI that performs like this one showing next, again as an animated gif.

![](responsive-keyboard-demo-sm.gif)

## How Suspense Changes How You Implement Showing Data

### Without Suspense

Let's assume you have some kind of external data source such as a remote web service, or even just data from a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) Server. Without using Suspense (which by the way, is still an option in React 18), you would programmatically fetch the data, then, check some data loading state, and finally, when that loading state indicates the data is fully retrieved, show the data in the UI. You app code probably looks something like this.

```JavaScript
function CityList() {
  const [data, setData] = useState();
  const [isLoading, setIsLoading] = useState(true);
  useEffect(() => {
    setIsLoading(true);
    fetchCities().then((records) => {
      setData(records);
      setIsLoading(false);
    });
  }, []);

  if (isLoading) return <div>Loading cities...</div>;

  return (
    <ul>
      {data.map(({id, name}) => {
        return <li key={id}>{name}</li>;
      })}
    </ul>
  );
}
```

### With Suspense

If you were to do the same thing with React 18 Suspense you would do this. Instead of doing this in a single component, you first create another component that wraps the rendering part of this component inside (as children that is) a new Suspense Element. Then, as an attribute to the Suspense element, you'd pass a fallback attribute, and that attribute would get assigned to a fallback UI that gets displayed when the data the component is not available to render (meaning it has not been completely returned from an external source).  Roughly, the code might now look something like this.

```JavaScript
function CityListSuspenseWrapper() {
  const specialPromiseResource = getSpecialPromiseTofetchCities();
  return (
    <Suspense fallback={<div>Loading cities...</div>}>
      <CityList resource={specialPromiseResource} />
    </Suspense>
  )
}

function CityList({resource}) {
  const data = resource.readData();
  return (
    <ul>
      {data.map(({id, name}) => {
        return <li key={id}>{name}</li>;
      })}
    </ul>
  );
}
```

What's happening here, is the line of code `const specialPromiseResource = getSpecialPromiseTofetchCities()` is executed outside of our Suspense component, and it essentially requests the data from the outside service or REST call. The call then returns immediately, and most importantly, returns with only a special resource reference to a promise about this particular data retrieval call.

Looking into our updated React CityList component, notice that we no longer have our programmatic calls that get executed after the page renders, that is the code passed into `useEffect`, but instead we simply call `resource.readData()` instead and assume, for the sake of this component, that when data is returned, we can immediately render that data without being concerned about any loading, or even an error state.

What makes this work, is that the code inside our `specialPromiseResource` method is using special capabilities that cause our `CityList` component to render, when, and only when, the promise to get the city list data is complete.  You can think about the call inside the `CityList` component, that is the line `const data = resource.readData()`, as helping the new Concurrent Mode Suspense feature figure out if it's time to render the city list.

The beauty of this code, is that **building components now to render data is much simpler** for us, and we don't need to worry about tracking loading state changes like we had to do before we had Suspense.

You might be thinking that this doesn't seem that different then code we build prior to Suspense and Concurrent Mode.  That's probably true, but in this example, our particular React component only has to deal with one outside call to a single data source. Think about the case of a complex web site where a page may include data from many different sources.  Before any rendering occurs, the idea is that we setup all our special promise calls to get data and then as the page renders and these external calls complete, different parts of the page render. With our new Suspense implementation, the React Concurrent Mode rendering engine can **optimally figure out how to re-render the page** based on what data arrives first, while at the same time, only rendering what is necessary to our browser DOM.

## Imperative verses Declarative

From a user’s perspective regardless of whether or not we use Suspense’s Concurrent Mode in our apps, result in the browser will basically be the same.  That is, the UI first displays a loading message, followed shortly be the data being displayed.

The difference to us though, as developers and designers, is significant.  Without Suspense we have to programmatically track our loading and error states of all of our components. For a single external data call, like we have in our example above with our list of cities, the problem is not hard.  However, when our component hierarchies get more complex, and we have multiple dependencies, tracking the states can get really difficult and confusing.  This type of code where we explicitly track and act on things like state values, we **typically refer to as imperative**, meaning we state exactly what we want, in the order we want it, with all the required conditionals along the way.

**Declarative** code, on the other hand, is typically easier to write, as it means instead of having to figure out what order everything should be done, we state (or declare) our intention of what we want, and we let the application, or in our case, the new Concurrent Mode React rendering engine, figure it out and do it for us.

That is, before Concurrent Mode and Suspense the code above with `useEffect`, now has element declarations like `<Suspense fallback={<div>Loading cities...</div>}>`.


## The Elephant in the Room

The expression "The Elephant in the Room" is used when there is something important that needs to be discussed, everyone is aware of it and no one wants to talk about it.

In the case of React 18 Concurrent Mode and Suspense, is there an elephant in the room at the moment considering Suspense is in full production release? Unless you use the very opinionated and somewhat advanced programming API [Relay](https://relay.dev/), you cannot use Suspense and the new Concurrent Rendering mode and expect them to work well in production.

The reason the Facebook React team includes Suspense and Concurrent Mode in their latest release is that **it is absolutely 100% production ready if you use Relay**, and as proof of that, it is currently what is driving probably the highest volume and likely the most used site on the internet, facebook.com. If you really want the benefits of Suspense and Concurrent Mode, you can build your apps with Relay.

If, however, you are like most of us developers and using one of the other data access libraries like [SWR](https://swr.vercel.app/), [React-Query](https://react-query.tanstack.com/), or even [ApolloGraphQL](https://www.apollographql.com/), you will not be able to use Suspense. The React team has made clear that their intention is to make Suspense work with our favorite data access API's, but for now, we just need to wait and make sure we are ready. Once the React team figures out a good way to integrate data into Suspense, and the third party data libraries declare the libraries production ready, everyone will be able to migrate to Suspense.

## How to Learn to Program Suspense Now

The React team has published documentation that includes a very good example of how to use Suspense that I used in my sample code. Thinking about the pseudo code above where I call `const specialPromiseResource = getSpecialPromiseTofetchCities()` to get our special promise resource to be used in our Suspense wrapped component, as well as the call `const data = resource.readData()` in the data rendering component itself, it creates a working example with a "not for production" sample implementation of the necessary resources.

If you are thinking, maybe you can use this code, and cut and paste it to your own apps and release them to production, you should really think again.  You'll find yourself copying these lines of commented out code along with the rest.

```JavaScript
// Suspense integrations like Relay implement
// a contract like this to integrate with React.
// Real implementations can be significantly more complex.
// Don't copy-paste this into your project!
```

That said though, if you do follow the example code and build some of your own applications following the same pattern, you will be well on your way to using Suspense and the new Concurrent Mode rendering engine in your apps when data libraries are available that do implement data fetching with these new features.

## What's Ahead from Pluralsight

As of now, there are no courses in the Pluralsight library that teach React and Suspense programming. Several months ago, I did release a course titled [React 18 First Look](https://app.pluralsight.com/library/courses/react-18-first-look). It does talk about Suspense, though it's not entirely consistent with what was released in the final release of React 18. There is a lot of good stuff there.

At the moment, I'm completing work on a new course to replace the First Look one titled "What's New in React 18", which will be much more up to date, as it's done based on the final React 18 release and not a preview release.

It includes a significantly expanded example of using Suspense, as well as a more flexible implementation of the experimental library used for data and Suspense.  For example, the component I build to render city lists and city details using Suspense now looks like this.

```JavaScript
function CityListAndDetail() {
  return (
    <CityListStoreProvider initialDisplayCount={5}>
      <div className="container">
        <h2>With Suspense</h2>
        <CityDisplayCount />
        <div className="row">
          <Suspense fallback={<div>Loading CityList...</div>}>
            <CityList>
              <Suspense fallback={<div>Loading CityDetail...</div>}>
                <CityDetail />
              </Suspense>
            </CityList>
          </Suspense>
        </div>
      </div>
    </CityListStoreProvider>
  );
}
```

To get an idea of now what the CityList component looks like, I now have this.

```JavaScript
function CityList() {
  const { getCities } = useContext(CityListStoreContext);
  const cities = getCities();
  return (
    <ul className="list-group">
      <li className="list-group-item">City list</li>
      {cities.map(({ id, name }) => {
        return (
          <Fragment key={id}>
            <CityButton city={name} />
          </Fragment>
        );
      })}
    </ul>
  );
}
```

This is somewhat simplified of the actual implementation in the upcoming course, but I'm sure you get the idea. We use the React Context feature to store the special promise resource that loads all our page promises before the page renders, then, using that context, nested deep in the component hierarchy, we fetch data to be used only when the Concurrent Mode rendering engine expects the component to render.

If you want to be notified when this course, "What's New In React 18" is released, head over to my [Author's profile](https://app.pluralsight.com/profile/author/peter-kellner) here in Pluralsight and click the "Follow" button and you'll get an email as soon as it comes out.

![](peterkellner-author-profile-400.png)

You can also come back to this post anytime, and I'll make sure a note gets put at the top indicating the new course is out with a link to it.

Thanks for reading, -Peter Kellner, Pluralsight author
