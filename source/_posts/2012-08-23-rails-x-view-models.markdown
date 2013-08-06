---
layout: post
title: "Rails x View Models"
date: 2012-08-23 10:38
comments: true
published: false
categories: [view model, rails, arquitetura]
---

class MyViewModel
  extend ActiveModel::Naming 
  include ActiveModel::Conversion
  def persisted?
    false
  end
end