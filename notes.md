

# Notes
## MutationObserver

Reference : [MDN-docs](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver)

The MutationObserver API in JavaScript lets you watch for changes in the DOM(Document Object Model),such as :
- Adding or removing elements.
- Changing attributes of elements.
- Modifying text content.

#### Let's consider a basic example :

Suppose we want to monitor when new elements are added inside a <div> container.

```html
<!DOCTYPE html>
<html>
    <head>
        <title>MutationObserver Example</title>
    </head>
    <body>
        <div id = "container">
            <p>Exisiting paragraph</p>
        </div>
        <button
            onclick="addElement()"
        >
            Add element
        </button>

        <script>

            const targetNode = document.getElementById("container");

            function callback(mutationList, observer){
                mutationList.forEach( (mutation) => {
                    console.log("Mutation detected : ", mutation);
                });
            }

            const observer = new MutationObserver(callback);

            const config = { 
                childList : true,
                subtree : true
            };

            observer.observe(targetNode, config);

            function addElement(){
                let newElement = document.createElement("p");
                newElement.textContent = "New paragraph added!";
                targetNode.appendChild(newElement);
            }
        </script>
    </body>
</html>
```

**Explanation of Code:**

1. We select the `<div id="container">` element as our target.
2. We create a `MutationObserver` instance with a callback function that logs detected changes.
3. The observer starts watching `container` for changes.
4. When the user clicks the "Add Element" button, a new `<p>` element is added.
5. The observer detects the change and logs it in the console.

## Methods provided by MutationObserver API :

The MutationObserver provides three main methods :

1. `observe( targetNode, config )` : Starts observing changes in the specified target element.

    **Parameters :**
    - targetNode : The DOM element to observe.
    - config : An object defining what types of changes to watch for.

2. `disconnect()` : Stops the observer from watching changes.

3. `takeRecords()` : Returns an array of mutation records that have been detected but not yet processed.
This is useful when you want to manually handle pending mutations instead of waiting for the callback function to be executed.

### What is `config` object ?
The `config` object in `MutationObserver.observe(target, config)` specifies what types of DOM mutations you want to track.

### Available config options :

| Option | Type | Description |
|--------|------|-------------|
|childList|	boolean|	Observes changes to child elements (added/removed).|
|attributes|	boolean|	Observes changes to attributes.|
|attributeFilter|	string[]|	List of specific attributes to observe (e.g., ['class', 'id']).|
|attributeOldValue|	boolean|	Stores the old attribute value before the change.|
|characterData|	boolean|	Observes changes to text content inside elements.|
|characterDataOldValue|	boolean|	Stores the old text content before the change.|
|subtree|	boolean|	Observes all descendants of the target element.|

In above example we have used `const config = {childList : true,subtree : true};`
- `childList: true` = This means that the observer will watch for changes to the child elements (i.e., additions or removals of child nodes) of the target node.
- `subtree: true` = This tells the observer to also watch for changes to all descendant nodes, not just the direct children of the target node.

## The callback method for MutationObserver 

The callback function for a MutationObserver receives two parameters :

1. **mutationList (Required) :** An array of MutationRecord objects, each describing a detected change in the observed element(s).
2. **observer (Optional) :** The MutationObserver instance that triggered the callback.


## Structure of a MutationRecord :

### What does the `mutationList` contains?

The Mutation List (mutationsList) is an array of MutationRecord objects that provides details about the detected changes.

Every time a mutation occurs, the MutationObserver callback receives this list of mutations, where each mutation contains information such as:

- What type of change occurred (childList, attributes, or characterData).
- Which elements were added or removed.
- Which attribute changed and its old value.
- If text content changed.

Each mutation in `mutationList` has the following properties :

| Property | Description |
|----------|-------------|
|type|	Type of mutation (attributes, childList, or characterData).|
|target|	The element that was mutated.|
|addedNodes|	A NodeList of elements that were added.|
|removedNodes|	A NodeList of elements that were removed.|
|previousSibling|	The previous sibling of the added/removed node(s).|
|nextSibling|	The next sibling of the added/removed node(s).|
|attributeName|	Name of the changed attribute (only for attributes mutation).|
|oldValue|	The previous value of the attribute or text content (if attributeOldValue: true or characterDataOldValue: true is set in the config).|

**Consider below example which will observer different types of mutations and log them.**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>MutationObserver Example</title>
</head>
<body>
    <div id="container">
        <p>Original text</p>
    </div>
    <button onclick="addElement()">Add Element</button>
    <button onclick="changeText()">Change Text</button>
    <button onclick="changeAttribute()">Change Attribute</button>

    <script>
        // Select the target node
        const targetNode = document.getElementById("container");

        // Callback function for the observer
        const callback = (mutationsList, observer) => {
            mutationsList.forEach(mutation => {
                console.log("Mutation Type:", mutation.type);
                console.log("Target Element:", mutation.target);

                if (mutation.type === "childList") {
                    console.log("Added Nodes:", mutation.addedNodes);
                    console.log("Removed Nodes:", mutation.removedNodes);
                } else if (mutation.type === "attributes") {
                    console.log(`Attribute changed: ${mutation.attributeName}`);
                    console.log(`Old value: ${mutation.oldValue}`);
                } else if (mutation.type === "characterData") {
                    console.log(`Old text content: ${mutation.oldValue}`);
                }
            });
        };

        // Create a MutationObserver instance
        const observer = new MutationObserver(callback);

        // Configuration options
        const config = {
            childList: true,      // Detect adding/removing elements
            attributes: true,     // Detect attribute changes
            characterData: true,  // Detect text content changes
            subtree: true,        // Observe all descendant nodes
            attributeOldValue: true, // Store old attribute values
            characterDataOldValue: true // Store old text content
        };

        // Start observing the target node
        observer.observe(targetNode, config);

        // Functions to trigger mutations
        function addElement() {
            let newElement = document.createElement("p");
            newElement.textContent = "New paragraph added!";
            targetNode.appendChild(newElement);
        }

        function changeText() {
            let p = targetNode.querySelector("p");
            if (p) p.textContent = "Text changed!";
        }

        function changeAttribute() {
            targetNode.setAttribute("data-example", "changed");
        }
    </script>
</body>
</html>
```


## How MutationObserver.observe() Works - The Flow
Once you call `.observe()`, the `MutationObserver` starts watching for changes in the **specified element**. The `callback` function is triggered only when a **mutation occurs in the observed element**, not continuously in a loop.

**Flow of Execution :**

1. You create a `MutationObserver` instance and pass a callback function.
2. You call `.observe(targetNode, config)` to start watching for changes.
3. Whenever a **mutation occurs** (e.g., a child is added, text is changed, or an attribute is modified), the **callback function is triggered**.
4. The callback function receives a list of `MutationRecord` objects, which describe what changed.
5. This continues until you manually stop the observer using `.disconnect()`.



# How MutationObserver will help us in our tool?

After analyzing in the inspect element of the youtube's webpage we can come to idea of media-player.
This media-player plays the videos we select each time.

So this media-player can be our parent element which plays multiple videos inside it.Similarly it also plays the advertisements in same media-player.


Consider the below code which observes for this media-player to be loaded on the web-page at first.

```javascript

const bodyObserver = new MutationObserver((mutations, observer) => {
    for (const mutation of mutations) {
        if (mutation.type === 'childList') {
            mutation.addedNodes.forEach(node => {
                if (node.nodeType === Node.ELEMENT_NODE && node.id === 'movie_player') {
                    console.log('Observing movie_player element...')
                }
            });
        }
    }
});
```

In the above code we see for the `movie_player` node when it gets added on the youtube's webpage.Once we find it has been loaded console logs the statement.

Cool! Our job has not yet finished.Main target is the skipping of the ads that play inside this movie player.

Consider the further code which will trigger our next observer from here.

```javascript

const moviePlayerObserver = new MutationObserver(function (mutations, observer) {
    for (const mutation of mutations) {
        if (mutation.type === 'attributes' && mutation.attributeName === 'class') {
            const target = mutation.target;
            if (target.classList.contains('ad-showing')) {
                var adVideo = document.getElementsByClassName('video-stream html5-main-video')[0];
                if (adVideo !== undefined && !isNaN(adVideo.duration) && adVideo.currentTime !== adVideo.duration) {
                    console.debug('Found ad. Fast forwarding to the end.');
                    adVideo.currentTime = adVideo.duration;
                }
                adSkipButton = document.getElementsByClassName('ytp-ad-skip-button-modern ytp-button')[0]
                if (adSkipButton !== undefined) {
                    console.debug('Found Skip Ad button. Clicking it.');
                    adSkipButton.click();
                }
            }
        }
    }
});

const bodyObserver = new MutationObserver((mutations, observer) => {
    for (const mutation of mutations) {
        if (mutation.type === 'childList') {
            mutation.addedNodes.forEach(node => {
                if (node.nodeType === Node.ELEMENT_NODE && node.id === 'movie_player') {
                    console.debug('Observing movie_player element...')


                    ///////////////movie_player found //////////////////
                    const config = { attributes: true, attributeFilter: ['class'], subtree: true };
                    moviePlayerObserver.observe(node, config);
                }
            });
        }
    }
});

```

As soon as the `movie_player` is loaded it plays either the ads or the selected video.To observe further what is being played inside the movie_player we trigger another MutationObserver from here which will now observe for the ads.

Once we find that the ad is being played inside this movie_player we can fetch the duration of the ad and seek forward to the duration of add which is indirectly skipping the advertisement.(This is only in the case if there is no `skip` button provided).

In case of `skip button` being displayed on the movie player we can simply click it using `adSkipButton.click();` this will automatically skip the ad.

