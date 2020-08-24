---
title: "How To Reference Existing Association Data With Rails form_for "
layout: post
permalink: /how-to-reference-existing-association-records-with-rails-form-for/
published: 17 February 2016
category: instagram tech
feature_image: /assets/img/posts/do-it.jpeg
description: How to refence separate model association data with Rails form_for form method
id: 15
---
<sub class='blog-date'>{{ page.published }}</sub>

This is a techy post as I'm going to start sharing some of the day to day things that I uncover while working on client projects in the hope that it helps other developers.

This is much easier that writing full series on complete project builds like I have in the past!

---

### The Challenge

In a recent client project I wanted to create a form which `accepts_nested_attributes_for` a model association BUT I wanted to display ALL of the available options to the user which are pre-existing records themselves.

Let me explain with the actual example.

In this Rails project, I have `material_summaries` that `have_many :materials`.

I also have `summary_prices` that `belong_to :material_summary` and which also `has_many :prices` which match to materials!

In my form for a new or existing `summary_price`, I wanted to pre-populate a list of *potentially available* `prices` for the `summary_price` by referencing the already saved `materials`.

(Let me know if this could be explained better in the comments)

The traditional way that was explained very nicely in this (RailsCast episode)[] is where the user adds individual groups of form fields for each new association using Rail's `fields_for` form helper BUT I wanted to show the user ALL of the available options at once (which we know from the list of created `materials`)

### The Solution

Using Rail's `fields_for` and the `new_record?` method, I was able to create the following solution. This is a simplified version of what I actually created and I'd recommend using helpers to tidy up the logic:

```ruby
<%= form_with(model: summary_price], local: true) do |form| %>
  <#% fields for the parent summary_price can go here %>
  <table class='table'>
    <thead>
      <tr>
        <th>Name</th>
        <th>Start Date</th>
        <th>Status</th>
        <th>Price</th>
      </tr>
    </thead>
    <tbody>
      <% @material_summary.materials.each do |material| %>
        <% if summary_price.new_record? %>
          <% form_object = form.object.send(:prices).klass.new(material_id: material.id, start_date: Date.today.at_beginning_of_month.next_month) %>
        <% else %>
          <% form_object = form.object.prices.find_by(material_id: material.id).present? ? form.object.prices.find_by(material_id: material.id) : form.object.send(:prices).klass.new(material_id: material.id, start_date: Date.today.at_beginning_of_month.next_month) %>
        <% end %>
        <%= form.fields_for(:prices, form_object, child_index: form_object.id.present? ? form_object.id : form_object.object_id) do |builder| %>
          <tr>
            <%= builder.hidden_field :material_id, value: builder.object.material_id %>
            <td>
              <%= material.combined_name %>
            </td>
            <td>
              <%= builder.date_field :start_date, class: 'form-control' %>
            </td>
            <td>
              <%= builder.select :status, options_for_select(Price.statuses.map { |k, _v| [k.humanize.capitalize, k] }), {}, class: 'form-control' %>
            </td>
            <td>
              <%= builder.number_field :amount, class: 'form-control', min: 0, step: 0.01, required: true %>
            </td>
          </tr>
        <% end %>
      <% end %>
    </tbody>
  </table>
  
  <%= form.submit class: 'btn btn-brand btn-upper btn-bold mt-3' %>
<% end %>
```

For each `material` in the `@material_summary` that is being referenced, we're creating a new `form_object` that either uses default data for a new `price` record OR we're finding the associated existing `price` record and using the data from that record for the form.

This solution does make a new DB call with each iteration of the `.each` method, so be careful if you're using this solution for very large amounts of data and where performance is crucial.

Have any questions? Let me know in the comments!