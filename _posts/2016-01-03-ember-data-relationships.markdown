---
layout: post
title:  "Ember Data Relationships"
date:   2016-01-03 20:00:39
categories: Ember Data
---
Relationships can be difficult and complex. Throw in some children and things get even harder. I'm talking about database relationships of course.

I kind of made a mistake by assuming that when saving a `hasMany` - `belongsTo` relationship in Ember data that both sides would record the relationship to the database.

e.g.,
{% highlight js %}
// models/car.js
export default DS.Model.extend({
	options: DS.hasMany('option', { async: true }),
	model: DS.attr('string')
});

// models/option.js
export default DS.Model.extend({
  car: DS.belongsTo('car', { async: true }),
  optionsText: DS.attr('string')
});
{% endhighlight %}

I thought would produce something like this,
{% highlight js %}
// car
{ car:
	{ id: 90001, make: "Ford", options:[ 10001, 10002 ] }
}

// option 10001
{ option:
	{ id: 10001, optionText: "Sunroof", car: 90001}
}
{% endhighlight %}

But actually the hasMany relationship is not serialized and persisted.

After spending some time trying to debug something that was not a bug, I found that this is the default behavior.

[EMBEDDEDRECORDSMIXIN DEFAULTS](http://emberjs.com/api/data/classes/DS.EmbeddedRecordsMixin.html#toc_embeddedrecordsmixin-defaults)

>If you do not overwrite attrs for a specific relationship, the EmbeddedRecordsMixin will behave in the following way:
BelongsTo: { serialize: 'id', deserialize: 'id' } HasMany: { serialize: false, deserialize: 'ids' }

Of course I could have just modified the serializer so that it behaved as I first expected, but after some thought I realized that it that way for a good reason.

My solution then was just to pull the data correctly in the route.
{% highlight js %}
model: function(param) {
    console.log(param.car_id);
    var car = this.store.find('car', param.car_id);
    return this.store.find('feature', {car: param.car_id}).then(function (features) {
    	car.features = features;
    	return car;
    });
}
{% endhighlight %}

I welcome any insight you may have.