# NoSpoil: a simple household food inventory management application
Ever forget something at the corner of your fridge until it smells? This app can track the Best Before Date of your food inventories, and help reduce the chance of food waste.

## Features
- User authentication
- Create and manage product categories
- Create and manage stock items under each product category
- Quickly glance food items that are due today, or about to expire within 5 days
- Quickly consume or spoil an item
- Track food spoil rate for each product category, try to reduce it as much as you can!
- Dark mode by design :)

## General application information
- Ruby version: 3.1.1
- Tested on Firefox 102.0 and Chrome 103.0.5060.134
- PostgreSQL version: 14.4

## Getting Started
Navigate to the project root folder and install the corresponding Ruby version and Gems:
```bash
bundle install
```

To initialize the database, run
```bash
createdb ns_db
```

Import seed data:
```bash
psql ns_db < ./data/db_dump.sql
```

Finally, start the app by running:
```bash
ruby app.rb
```
Now open the browser and visit `localhost:4567`, you should be welcomed by the Sign In page.

To log in, you can either create a new user, or use admin/admin for username/password.

## Requirements and where they are met in code
- *The application must use at least two kinds of related data where one of the data types is a collection of objects of the other type.*

  In this project, three entities are used: `users`, `products` and `items`. `products` and `items` has a `one-to-many` relationship, meaning that one product can have none to many items, while one item must belongs to one and only one product.

  Please refer to `./data/schema.sql` for how entities and attributes are set up. I also have created seperated files and classes for each entity:
  - `./lib/user.rb` contains classes and methods that relates to the `users` entity
  - `./lib/items.rb` contains classes and methods that relates to the `items` entity
  - `./lib/products.rb` contains classes and methods that relates to the `products` entity
- *The application must provide CRUD capabilities (create-read-update-delete)*

  CRUD capabilities are implemented for `items` and `products`. CR capabilities are implemented for `users`.

  Please refer to `./app.rb` for complete routes that relate to CRUD operations mentioned above. For CRUD methods/SQL Statements, please refer to files for each entities under the `lib` folder. (For example, you will find CRUD methods/SQL Statements for `items` in `./lib/items.rb`)

  In addition, logging is enabled in Sinatra so each database query will be shown in the terminal.

- *The page used to update a collection or object must have a unique URL.*

  In `./app.rb`, `line 81` shows the unique URL when editing an item; `line 131` shows the unique URL when editing a product.

- *When listing collections and objects, limit the amount of output per page to a maximum item count.*

  Each time an user requests a specific page of data, only the data for this page is fetched from the database. For example, when an user requests a page of `item` data:
  - `./lib/items.rb : line 60`: the SQL statement for fetching the data for the specific page
  - `./lib/helers.rb : line 121`: the method that calculates how many pages in total
  - `./lib/helers.rb : line 104`: the method that validate and handles non-existent page requests
  - `./views/stock.erb : line 71`: shows how pagination component is rendered

- *When listing collections or objects, sort the items consistently.*

  - `./lib/items.rb : line 67` shows how `items` are ordered
  - `./lib/products.rb : line 85` shows how `products` are ordered

- *Validate input data as needed. In particular, you must prevent SQL and JavaScript injection.*

  Most input validation are done client side by specifying the input type using `<input type="">`. However, some validation are done server side, ex. trying to create a user with the same username

  SQL injection is prevented by calling `exec_params` method for every SQL statements. The wrapper method `query` that implements it can be found in `./lib/dbconnection.rb : line 20`

  JavaScript injection is taken care of by enabling `HTML escape` in Sinatra, and using `<%== %>` in all `erb` files

- *Display appropriate flash error messages when the user does something incorrectly. Error messages should be displayed on the same page where they are raised and should be specific. If the page contains multiple inputs, you should preserve any valid data that the user has already entered.*

  All error messages can be found in `./lib/helpers.rb` and stored in `session[:error]`. For example, if an user tries to add a duplicate `product` with the same name, an error message will be shown. Please refer to `./app.rb : line 123` and `./lib/helpers.rb : line 87` for code implementation

  When an error happens, any valid data that the user has already entered is preserved. Please refer to `./views/add_product.erb : line 5` for one such code example.

- *Validate URL parameters such as ID numbers and query strings.*

  In this project, all `item_id`, `product_id` and `page_num` are validated. Please refer to `./helpers.rb : line 104-119` for details.

- *The application must require login authentication.*

  Login authentication is implemented, including following features:
  - Sign-up: `./lib/helpers.rb : line 59-71`
  - Sign-in: `./lib/helpers.rb : line 45-57`
  - Redirect user to the sign-in page if user is not signed in: `./app.rb : line 32` and `./lib/helpers.rb : line 73`
  - Redirect user to the previous URL after sign in: `./lib/helpers.rb : line 54 and 75`

## Hope you enjoy the app as much as I enjoy creating it!
