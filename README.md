Juissy is a minimal experimental JSON API client for Drupal.

Features:
Zero-configuration
Automatic pagination
Late-binding filter compiler
???
Example:
// A client only needs a base URL. It doesn't need to know anything else!
const client = new JuissyClient('http://jsonapi.test:8080');

// It's best to read the code beneath these comments, then fill in your gaps in
// understading with these comments.

// `client.all()` returns a Promise. You may specify a limit for number of
// resources to retrieve, sorting rules, and filters too! If no limit is given
// the client will *lazily* resolve every resource on the server!
  // The Promise returned by `client.all()` resolves to a feed.
    // You "consume" resources by specifying a function to run for every
    // resolved resource. This will run for every resource up to the given
    // maximum or until there are no more resources available.
      // `consume` itself returns a Promise that will resolve to either a
      // function or `false` if there are no more resources available.
          // The `more` function lets you increase the number of resources to be
          // resolved.
          // Once the maximum has been increased, you may consume the additional
          // resources.
            // While the second `consume` call is "nested" here for the sake of
            // example, you need not do the same. Just take care that `consume`
            // is not called again before the first `consume` has completed.

client.all('node--recipe', { limit: 3, sort: 'title' })
  .then(feed => {
    return feed.consume(print('Initial'))
      .then(more => {
        console.log(`There are ${more ? 'more' : 'no more'} resources!`);
        if (more) {
          more(10);
          feed
            .consume(print('Additional'))
            .then(evenMore => {
              console.log(`There are ${evenMore ? 'more' : 'no more'} resources!`);
            });
        }
      });
  })
  .catch(error => console.log('Error:', error));

// This will just print the title of every recieved resource.
const print = (label) => {
  return resource => console.log(`${label}:`, resource.attributes.title);
};
