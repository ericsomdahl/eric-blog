{:title "Cryogen Workflow"
 :layout :post
 :tags  ["cryogen", "leiningen"]}

### The github deployment plot thickens
In order to deploy my [Cryogen](http://cryogenweb.org) blog onto github.io, I had created 
two different repos

  * My Cryogen project
  * the github.io generated pages

After running 
```
lein ring server-headless 
```
in the cryogen project I would manually copy the generated files from the /resources/public/
folder to the root of the github.io repo.  I assumed I was missing something.

According to [this page](http://tangrammer.github.io/posts/02-12-2014-cryogen-and-github.html), 
this is basically it.  Very meh.

Perhaps a Leiningen plugin is in order.
