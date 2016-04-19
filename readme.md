# Error Handling

## Screencasts
- [Part 1/5](https://youtu.be/sHSsc7EGI0g)
- [Part 2/5](https://youtu.be/wKMuNDLDKFA)
- [Part 3/5](https://youtu.be/HZPTIpZUAnQ)
- [Part 4/5](https://youtu.be/yvNO_yoWcqE)
- [Part 5/5](https://youtu.be/hEKDLhZQOSA)

## Learning Objectives

- Explain the purpose of `flash` in Rails
- Compare and contrast `flash[:notice]` and `flash[:alert]`
- List three ActiveRecord methods that trigger validations
- Compare and contrast the validation helpers `confirmation`, `inclusion`, `exclusion`, `length`, `presence`, `uniqueness`, and `numericality`
- Compare and contrast the validation options `allow_nil`, `allow_blank`, and `on`
- Create a custom validation on an ActiveRecord model
- Explain the benefits of explicitly handling errors
- Produce and handle an error in Ruby using the keywords `begin`, `rescue`, and `raise`.

## Setup

Clone down the [errors and validations Tunr repo](https://github.com/ga-dc/tunr_rails_errors_validations).

## Lesson Sections

- [flash](flash.md)
- [validations](validations.md)
- [error_handling](error_handling.md)

# Sample Quiz Questions (5 min)

- What's the difference between `.create` and `.create!`?
- You see `ROLLBACK` in your Rails server log. What does that mean?
- What does it mean to let something "fail silently", and why is it considered bad?
- What's the difference between `begin` and `rescue`?
- What's the difference between an error and an exception?
