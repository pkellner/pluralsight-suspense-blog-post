# React 18 and Suspense

[React 18 has shipped](https://reactjs.org/blog/2022/03/29/react-v18.html) as of March 29th, 2022. The new Concurrent Mode feature Suspense is 100% production ready. However, unless you have the development resources necessary to use GraphQL with Relay, you likely will not be able to use Suspense in your React Apps. For now though, you should pay attention, and understand Suspense if you are a React developer. In this post, you'll learn what you will eventually need to **integrate Suspense into your React app**.

## What is Suspense and Do I Need It?

Suspense is the first feature released by the React that takes advantage of the new rendering engine built into React 18 that when running, is referred to as **Concurrent Mode**. Benefits of using it include apps having better **more responsive UIs** while using less browser resources, as well as giving developers and designers a **more intuitive API to work with**. 

Suspense has been in the making by the Facebook React team for over 3 years. It fundamentally changes how React figures out what to render on web page rendered by your React app based on component state changes. That is, if there is any data (also known as state date), on your React's app web page that changes, and that app does not require a full page re-render from the server, it's the rendering engine inside of React that takes care of that.

The main difference between Concurrent Mode, and the React rendering engine prior, is that when a web page updates based on some interruption, like say a text box is used as a filter to list of items, without Concurrent Mode, it's likely that all the items in the list will get updated by React causing both the page to feel sluggish to the user, as well as use a lot of your computer resources to make it work at all. If you can view the animated gif below, this is what you might see.

![](not-responsive-keyboard-demo-sm.gif)

If, on the same computer, you were using an app the implemented Concurrent Mode properly, meaning that as the user typed into the search box, only what was necessary in the DOM was updated, you might see a UI that performs like this one showing next, again as an animated gif.

![](responsive-keyboard-demo-sm.gif)

## How Suspense Changes How You Implement Showing Data

### Without Suspense

Let's assume you have some kind of external data source such as a remote web service, or even just data from a REST Server. Without using Suspense (which by the way, is still an option in React 18), you would programmatically fetch the data, then, check some data loading state, then, when that loading state indicates the data is fully retrieved, show the data in the UI. You app code probably looks roughly as follows.

```JavaScript
function CityList() {
  const [data, setData] = useState([]);
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
      {data.map((rec) => {
        return <li key={rec.id}>{rec.city}</li>;
      })}
    </ul>
  );
}
```

### With Suspense

If you were to do the same thing with React 18 Suspense, instead of doing this in a single component, instead you first, create another component that Wraps the rendering part of this component inside (as children that is) a new Suspense Element. Then, as an attribute to the Suspense element, you'd pass a fallback attribute, and that attribute would get assigned to a fallback UI that gets displayed when the data the component is to render, is ready.  Roughly, the code might now look something like this.

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
      {data.map((rec) => {
        return <li key={rec.id}>{rec.city}</li>;
      })}
    </ul>
  );
}
```

What's happening here, is the line of code `const specialPromiseResource = getSpecialPromiseTofetchCities()` is executed outside of our Suspense component, and it essentially requests the data from the outside service or REST call, then returns immediately, and most importantly, returns with only a special resource reference to a promise about this particular data retrieval call.

Then looking into our updated React CityList component, notice that we no longer have our programmatic calls that get executed after the page renders, that the code passed into `useEffect`, but instead we simply call `resource.readData()` instead and assume, for the sake of this component, that when data is returned, we can immediately render that data without being concerned about any loading, or even an error state. 

What makes this code work, is that the code, inside our `specialPromiseResource` method, is returning to use special capabilities that cause our `CityList` component to render, when, and only when the promise to get the city list data is complete.  You can think about the call inside the `CityList` component, that is the line `const data = resource.readData()` as helping React, and the new Concurrent Mode Suspense feature figure out if it's time to render the city list.

The beauty of this code, is that **building components now to render data is much simpler** for us, and we don't need to worry about tracking loading state changes like we had to do before we had Suspense.

You might be thinking that this doesn't seem that different then code we build prior to Suspense and Concurrent Mode.  That's probably true, but in this example, our particular React component only has to deal with one outside call to a single data source. Think about the case of a complex web site where a page may include data from lots of different sources.  Before any rendering occurs, the idea now, is that we setup all our special promise calls to get data, and then, as the page renders, and these external calls complete, different parts of the page may render. With our new Suspense implementation, the React Concurrent Mode rendering engine can **optimally figure out how to re-render the page** based on what data arrives first, while at the same time, only rendering what is necessary to our browser DOM.

## Imperative verses Declarative

From a users perspective regardless of whether or not we use Suspense, based on Concurrent Mode, in our apps, result in the browser will basically be the same.  That is, the UI first displays a loading message, followed shortly be the data being displayed.

The difference to us though, as developers and designers, is significant.  Without Suspense we have to programmatically track our loading and error states of all of our components. For a single external data call, like we have in our example above with our list of cities, the problem is not hard.  However, when our component hierarchies get more complex, and we have multiple dependencies, tracking the states can get really hard and confusing.  This type of code we **typically we refer to as imperative**, meaning we have state exactly what we want, in the order we want it, with all the required conditionals along the way.

**Declarative** code, on the other hand, is typically easier to write, as it means instead of having to figure out what order everything should done, we more state (or declare) what we want, and we let the application, or in our case, the new Concurrent Mode React rendering engine, figure it out.

That is, basically the difference between the code above with `useEffect` and was previous to Concurrent Mode and Suspense, compared to the code after that, that has element declarations like `<Suspense fallback={<div>Loading cities...</div>}>`.


## The Elephant in the Room

The expression "The Elephant in the Room" is used when there is something that needs to be discussed, and no one really wants to talk about it, but usually, it's pretty important.

In the case of React 18, Concurrent Mode and Suspense, the elephant is that at the moment, Suspense is in full production release, but unless you use the very opinionated and somewhat advanced programming API [relay](https://relay.dev/), you can not take advantage of Suspense and the new Concurrent Rendering mode now in your React Apps.

The good news though, and the reason the Facebook React team include Suspense and Concurrent Mode in their latest release, is that it is absolutely 100% production ready, and as proof of that is currently what is driving, probably the highest volume and likely the most used site on the internet, facebook.com. If you really want the benefits of Suspense and Concurrent Mode, you also can build your apps with Relay right now, just like Facebook, and release them to production.

If however, you are like most of us developers, and using one of the other data access libraries like useSwr, react-query, or even ApolloGraphQL, for now, you'll not be able to use Suspense in your React apps. The React team has made clear that their intention is for us to at some point be able to use Suspense with our favorite data access API's, but for now, we just need to wait and make sure we are ready, once the React team figure it out and the third party data libraries declare the libraries production ready.

## How to Learn to Program Suspense Now

A while back, the React team published documentation, including a very good example of how to use Suspense in your app. Thinking about the pseudo code above where I call `const specialPromiseResource = getSpecialPromiseTofetchCities()` to get our special promise resource to be used in our Suspense wrapped component, as well as the call `const data = resource.readData()` in the data rendering component itself, this documentation creates a working example with a "not for production" sample implementation of the necessary resources.  

If you are thinking, maybe you can use this code, and cut and paste it to your own apps and release them to production, you should really think again.  You'll find yourself copying these lines of commented out code along with the rest.

```JavaScript
// Suspense integrations like Relay implement
// a contract like this to integrate with React.
// Real implementations can be significantly more complex.
// Don't copy-paste this into your project!
```

That said though, if you do follow the example code and build some of your own applications following the same pattern, you will be well on your way to using Suspense and the new Concurrent Mode rendering engine in your apps when data libraries are available that do implement data fetching with these new features.

## What's Ahead from Pluralsight

As of now, there are no courses in the Pluralsight library that teach React and Suspense programming. Several months ago, I did release a course titled [React 18 First Look](https://app.pluralsight.com/library/courses/react-18-first-look). It does talk about Suspense, though it's not entirely consistent with what was released in the final release of React 18. There is a lot of good stuff there though.

At the moment, I'm completing work on a new course to replace the First Look one titled "What's New in React 18", which will be much more up to date, as it's done based on the final React 18 release and not a preview release.

It includes a significantly expanded example of using Suspense, as well as a more flexable implementation of the experimental library used for data and Suspense.  For example, the component I build to render city lists and city details using Suspense now looks like this.

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

To get an idea of now what the CityList component looks like, I will have this.

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

You can also come back to this article anytime, and I'll make sure a note gets posted at the top indicating the new course is out with a link to it.

Thanks for reading, -Peter Kellner, Pluralsight author.























