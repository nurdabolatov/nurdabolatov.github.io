---
layout: post
title: Trie Search in JavaScript
---

Today, we are going to implement a local prefix search functionality using the [Trie](https://en.wikipedia.org/wiki/Trie) data structure in JavaScript. We will analyze this solution and compare it against naive search.

## Problem Statement

We are given a list of text snippets and a search keyword. Our task is to efficiently find a text snippet with the word that starts with the specified search keyword. For example, let's take this array as an input list:

```javascript
const textSnippets = [
    'hello people',
    'Welcome to my World!',
    'Test your search results',
    'This is another test',
];
```

If a user enters `te`, we have to return `Test your search results` and `This is another test`.

A good example for this use case can be a to-do list app where you need to find a to-do item containing a particular word.

## Groundwork

First of all, we need to define how MVP should look like. We can start with a simple text field and `ul` element that contains search results:

```html
<input id="search" placeholder="Search" onkeyup="search()"/>

<ul id="results"></ul>
```

Now we need a helper function to render the results. This function accepts iterable data structure that consists of text snippet indices and renders them in the `ul` element:

```javascript
const renderSearchResults = (results) => {
    const ul = document.getElementById('results');
    ul.innerHTML = '';
    for (const result of results) {
        const li = document.createElement('li');
        const text = document.createTextNode(textSnippets[result]);
        li.appendChild(text);
        ul.appendChild(li);
    }
};
```

## Naive Search

At this point, we are ready to plug in any search algorithm. Let's implement the naive search. We iterate over all text snippets and search for a word that starts with the search keyword. Then add all matches to a set and render:

```javascript
const search = () => {
    const input = document.getElementById('search');
    const value = input.value.toLowerCase();
    const results = new Set();
    for (let i = 0; i < textSnippets.length; i++) {
        const words = textSnippets[i].split(' ');
        for (const word of words) {
            if (word.toLowerCase().startsWith(value)) {
                results.add(i);
                break;
            }
        }
    }
    renderSearchResults(results);
};
```

In the worst-case scenario (when there is no matching text snippet), this algorithm reads everything in `textSnippets` on each search request. However, in terms of memory, we only allocate as much memory as the longest snippet for `words` and a set of a size equivalent to the number of matches.

## Trie Search

When using the Trie we do need to read everything in `textSnippets` but we do it only once. Before we jump to that logic, let's break down how our data structure will look like.

For simplicity, I am going to use `['a test', 'the test']` as `textSnippets`. The graph for this data becomes:

```
root
┌──────┐   a   ┌─────┐
│  []  ├───────► [0] │
└───┬──┘       └─────┘
    │t
┌───▼────┐ h   ┌─────┐
│ [0, 1] ├─────► [1] │
└───┬────┘     └──┬──┘
    │e            │e
┌───▼────┐     ┌──▼──┐
│ [0, 1] │     │ [1] │
└───┬────┘     └─────┘
    │s
┌───▼────┐
│ [0, 1] │
└───┬────┘
    │t
┌───▼────┐
│ [0, 1] │
└────────┘
```

Here every node contains the set of indices that match the search request. For example, if the search keyword is `th`, starting from root node we take branch `t` then `h` which contains `[1]`. That means `textSnippets[1]` is our match - `the test`.

We are going to use a map to store our tree. For the example above the map is:

```json
{
    "a": {
        "results": [0]
    },
    "t": {
        "h": {
            "e": {
                "results": [1]
            },
            "results": [1]
        },
        "e": {
            "s": {
                "t": {
                    "results": [0, 1]
                },
                "results": [0, 1]
            },
            "results": [0, 1]
        },
        "results": [0, 1]
    },
    "results": []
}
```

Here is the code that builds the Trie:

```javascript
// This is to create an empty node in our graph
const createNode = () => {
    const node = new Map();
    node.set('results', new Set());
    return node;
};

const root = createNode();

const buildTrie = () => {
    // This function adds the word to the Trie and
    // the index to results field while traversing the tree.
    // Here the index is the index of text snippet that 
    // contains the word.
    const processWord = (index, word) => {
        let current = root;
        for (const letter of word.toLowerCase()) {
            // Create a new node if the path doesn't exist
            if (!current.has(letter)) {
                current.set(letter, createNode());
            }
            // Follow the path
            current = current.get(letter);
            // Add the index to results field 
            // in every node on the path
            current.get('results').add(index);
        }
    }

    // We break down every text snippet into words
    // and add every word into the Trie
    const processTextSnippet = (index, textSnippet) => {
        words = textSnippet.split(' ');
        for (const word of words) {
            processWord(index, word);
        }
    }

    for (let i = 0; i < textSnippets.length; i++) {
        processTextSnippet(i, textSnippets[i]);
    }
};

// We need to run this once when the application is loaded
buildTrie();
```

Now that we figured out how the data structure looks like it should be intuitive to perform the search. We will just follow the letters in the map while iteratiring over keyword letters. When we reach the end, we pass the `results` field to `renderSearchResults` and profit:

```javascript
const search = () => {
    const input = document.getElementById('search');
    let current = root;
    for (const letter of input.value.toLowerCase()) {
        // If the path doesn't exist, we return an empty result
        if (!current.has(letter)) {
            renderSearchResults(new Set());
            return;
        }
        current = current.get(letter);
    }
    renderSearchResults(current.get('results'));
};
```

## Conclusion

You may have already noticed that we only need to iterate over the keyword to get the search results. This is a huge advantage of using Trie in terms of time complexity compared to a naive solution. However, depending on the number of distinct words we are allocating a lot of memory to store our tree.

Trie is one of the many search algorithms you can take advantage of to perform a quick search. Never stop learning and discovering other alternatives. Let me know if you spot any bugs, want to share alternative solutions, or would like to improve this solution.

*Complete implementation available in [this GitHub Gist](https://gist.github.com/nurdabolatov/68e689a716c0b4d54454432726d5eefd)*