---
layout:     post
title:      Nested Models in Rails Forms
date:       2017-10-15 20:00:00
author:     Yury Voloshin
summary:    Introduction to creating Rails forms with nested models
categories: Ruby on Rails
---
The two years since my completion of [Firehose](http://www.thefirehoseproject.com)  have been an exciting time for me. While working as a PHP developer at [Columbia University](http://www.columbia.edu), I've completed one project and coming close to completing a second one. The first project was an application for tracking attendance at various events, where attendees scan their ID cards and their ID numbers are recorded in a database. The second project is a system for keeping track of students' travel abroad. The students fill out their travel plans online and a school administrator may approve, reject, or ask for more information. I've become comfortable with using the [Symfony](http://www.symfony.com) framework and with writing SQL queries, though there is still a lot to learn. In rare free moments, I've been working on a Rails app to keep up my Ruby and Rails skills. Ruby, and Rails, are fun to work with, and I'd hate to forget what I know of them. I have not been learning nearly as much of Ruby as I have of PHP, but working on this project helped keep what I already knew. 

The Rails app is called [RoutineTrack](http://www.routinetrack.com) and it is an exercise tracker. It allows the users to create routines, or sequences of exercises, which are then pre-filled into a table where the user saves the results of their exercises. It is different from most existing exercise tracking apps because it allows the users to create exercise routines prior to recording the results of their workouts. This way, there is no need to create a new list of completed exercises for every new workout. In this post, I'd like to talk about one aspect of this application, a form where the user creates a new exercise routine. An interesting part of this table is that the data entered by the user is saved into two different models. In other words, this is a form with nested models. It took me a while to figure out how to do this. 

First, a bit about the database schema. Here, I will talk about two models, `workout_types` and `exercise_types`. A workout_type is the name I'm using for an exercise routine. It has a name and a description, it belongs to a user, and it consists (`has_many`) of a number of exercises (called exercise_types). Each `exercise_type` has a name, a number of reps, a number of sets, a weight load, as well as the routine that it belongs to (as a foreign key). The ending "_type" was added to "workout" and "exercise" in order to distinguish the planned workouts and exercises from completed ones, which are called "workout" and "exercise". 

The controller method that creates a new `workout_type` looks like this:

```ruby
def new
	@workout_type = WorkoutType.new
	@exercise_types = Array.new(10) { @workout_type.exercise_types.build }
end
```

Here, the workout_type and the associated exercise_types (child objects of workout_type) are created. (Routines are set to have a maximum of 10 exercises.)

The exercise routine is created in a form that looks like this:
[![new routine form](https://i.imgur.com/7NC3on0.png)](https://i.imgur.com/7NC3on0.png)

This form was created using simple_form gem.The simple_form documentation has a [good description](https://github.com/plataformatec/simple_form/wiki/Nested-Models) of nested models. I followed the documentation to come up with the code below: 

```html
<%= simple_form_for (@workout_type) do |f| %>
	<%= f.input :type_name, label: 'What will you call this workout?' %>
	<br />
	<%= f.input :description, label: 'Add a brief description of this workout.', input_html: { class: 'mceEditor' } %>
	
	<p>Who should be able to see this workout?</p>
	<%= f.label :public, "All Users", :value => true %>
    	<%= f.radio_button :public, true, :checked => true %>
    	&nbsp;
    	<%= f.label :public, "Myself Only", :value => false %>
    	<%= f.radio_button :public, false %>
    
	<h3>Now enter up to 10 exercises:</h3> 
	<table class="table table-bordered table-striped table-responsive">
		<thead>
		    <tr>
		    	<th>Exercise Name</th>
			<th>Sets<br /><small><i>(assuming 1 set if left blank)</i></small></th>
		 	<th>Reps</th>
		  	<th>Load, lb or % max<br /><small><i>(leave blank if bodyweight)</i></small></th>
		  	<th>Link to instructions on how to perform this exercise</th>
		    </tr>
		  </thead>
		  <tbody>
		  	<%= f.fields_for :exercise_types, @exercise_types do |builder| %>
		      		<%= render 'exercise_fields_table', :f => builder %>
		    	<% end %>
		  </tbody>
	</table>
	<%= f.submit 'Create Workout Routine', :class=>"btn btn-primary", :id=>"routine_submit" %>
<% end %>
```

The helper method `simple_form_for` is used to specify the parent model, `workout_type`. The first half of the form contains fields for `workout_type`. The child objects of `workout_type`, `exercise_types`, are specified using the helper method `fields_for` that can be seen toward the bottom of the form. A partial form `exercise_fields_table` is rendered for each `exercise_type` that has been created in the controller's `new` method. And this is all that we need to do in the form itself to render fields belonging to associated models. 

There is a couple of things that need to be done outside of the form in order to make nested models work. We need to let Rails know that the form used to create a new `workout_type` object will also be used to create the child objects `exercise_types`. We do this by adding a `accepts_nested_attributes_for` statement in the `workout_type` model, like this:

```ruby
class WorkoutType < ActiveRecord::Base
	attr_accessor :exercise_type_attributes, :workout_type_id
	has_many :exercise_types, dependent: :destroy
	has_many :workouts
	belongs_to :user

	accepts_nested_attributes_for :exercise_types
end
```

Last (but not least), we need to add the nested attributes to the parameters of WorkoutType class, like this:

```ruby
def workout_type_params
	params.require(:workout_type).permit(:type_name, :public, :description, exercise_types_attributes: [:id, :name, :sets, :reps, :load, :url])
end
```

Note that the parameters of `:workout_type` do not include the `:id` of `:workout_type`, but the parameters of `exercise_type_attributes` do include the `:id`. There is a good reason for this difference. Imagine we edit a `:workout_type`, which has `exercise_types` associated with it. If the `exercise_type` did not have an `:id`, then upon submitting the edit form, Rails would have no way to determine which existing associated `exercise_types` need to be saved. It does the next best thing and creates new `exercise_types`. This is why, without `:id` in the nested parameters, Rails would double the number of associated instances with every edit. (I found this out the hard way.) More on this issue [here](https://stackoverflow.com/questions/18946479/ror-nested-attributes-produces-duplicates-when-edit) and [here](https://github.com/activeadmin/activeadmin/issues/2994).
