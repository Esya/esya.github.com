---
layout: post
title: "Benchmarking hasOwnProperty VS typeof"
category: posts
---
Today a question came to my mind. I've been wondering, out of `hasOwnProperty` and `typeof`, which one is the fastest? And to what extent? So I thought I would share the results of my small benchmark here. I used the excellent [BenchmarkJS module](http://benchmarkjs.com/) with a simple benchmark suite as follows : 


{% highlight javascript %}
var Benchmark = require('benchmark');

var suite = new Benchmark.Suite;
var obj = {prop: 'test'};

suite
.add('typeof not undefined',function() {
  ok = (typeof obj.prop === 'undefined');
})
.add('typeof undefined',function() {
  ok = (typeof obj.propnothere === 'undefined');
})
.add('Object#hasOwnProperty hit', function() {
  ok = obj.hasOwnProperty('prop');
})
.add('Object#hasproperty fail', function() {
  ok = obj.hasOwnProperty('propnothere');
})
.on('cycle', function(event) {
  console.log(String(event.target));
})
.on('error', function(event) {
  console.log(event.target.error);
})
// run async
.run({ 'async': true });
{% endhighlight %}

And here are the results :

```
typeof not undefined x 189,231,381 ops/sec ±12.42% (65 runs sampled)
typeof undefined x 159,348,486 ops/sec ±10.51% (69 runs sampled)
Object#hasOwnProperty hit x 21,246,750 ops/sec ±1.19% (86 runs sampled)
Object#hasOwnProperty fail x 20,544,326 ops/sec ±1.89% (85 runs sampled)
```

## Conclusion
To check whether or not an object contains a property, if you're greedy for performance, well, go for `typeof`. 
