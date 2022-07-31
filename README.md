# NoSpoil: a simple household food inventory management application
Ever forget something at the corner of your fridge until it smells? This app can track the Best Before Date for your food inventories, and help reduce the chance of food waste.

## Features
- User authentication
- Create and manage product categories
- Create and manage stock items under each product category
- Quickly glance food items that are expired, due today, or about to expire within 5 days
- Quickly consume or spoil an item
- Track food spoil rate for each product category, try to reduce it as much as you can!
- Responsive & Dark mode by design :)

## General application information
- Ruby version: 3.1.1
- Tested on Firefox 102.0 and Chrome 103.0.5060.134
- PostgreSQL version: 14.4

## Getting Started
Make sure specified Ruby and PostgreSQL versions are installed and correctly configured.

Navigate to the project root folder and install dependencies:
```shell
bundle install
```

To initialize the database, run
```shell
createdb ns_db
```

Import seed data:
```shell
psql ns_db < ./data/db_dump.sql
```

Finally, start the app by running:
```shell
ruby app.rb
```
Now open the browser and visit `localhost:4567`, you should be greeted by the Sign In page.

To log in, you can either create a new user, or use admin/admin for username/password.

## Requirements and where they are met in code
- *The application must use at least two kinds of related data where one of the data types is a collection of objects of the other type.*

  In this project, three entities are used: `users`, `products` and `items`. `products` and `items` has a `one-to-many` relationship, meaning that one product can have none to many items, while one item must belongs to one and only one product.

  Please refer to `./data/schema.sql` for how entities and attributes are set up. I also created seperated files and classes for each entity:
  - `./lib/user.rb` contains classes and methods that relates to the `users` entity
  - `./lib/items.rb` contains classes and methods that relates to the `items` entity
  - `./lib/products.rb` contains classes and methods that relates to the `products` entity
- *The application must provide CRUD capabilities (create-read-update-delete)*

  CRUD capabilities are implemented for `items` and `products`. CR capabilities are implemented for `users`.

  Please refer to `./app.rb` for complete routes that relate to CRUD operations mentioned above. For CRUD methods/SQL Statements, please refer to files for each entities under the `lib` folder. (For example, you will find CRUD methods/SQL Statements for `items` in `./lib/items.rb`)

  In addition, logging is enabled in Sinatra so each database query will be shown in the terminal.

- *The page used to update a collection or object must have a unique URL.*

  In `./app.rb`, `line 84` shows the unique URL when editing an item; `line 135` shows the unique URL when editing a product.

- *When listing collections and objects, limit the amount of output per page to a maximum item count.*

  Each time an user requests a specific page of data, only the data for this page is fetched from the database. For example, when an user requests a page of `item` data:
  - `./lib/items.rb : line 84`: the SQL statement for fetching the data for the specific page
  - `./lib/route_helpers.rb : line 84`: the method that calculates how many pages in total
  - `./lib/route_heleprs.rb : line 67`: the method that validate and handles non-existent page requests
  - `./views/stock.erb : line 71`: shows how pagination component is rendered

- *When listing collections or objects, sort the items consistently.*

  - `./lib/items.rb : line 91` shows how `items` are ordered
  - `./lib/products.rb : line 109` shows how `products` are ordered

- *Validate input data as needed. In particular, you must prevent SQL and JavaScript injection.*

  Most input validation are done client side by specifying the input type using `<input type="">`. However, some validation are done server side, ex. trying to create a user with the same username

  SQL injection is prevented by calling `exec_params` method for every SQL statements. The wrapper method `query` that implements it can be found in `./lib/dbconnection.rb : line 15`

  JavaScript injection is taken care of by enabling `HTML escape` in Sinatra, and using `<%== %>` in all `erb` files

- *Display appropriate flash error messages when the user does something incorrectly. Error messages should be displayed on the same page where they are raised and should be specific. If the page contains multiple inputs, you should preserve any valid data that the user has already entered.*

  All error messages can be found in `./lib/route_helpers.rb` and stored in `session[:error]`. For example, if an user tries to add a duplicate `product` with the same name, an error message will be shown. Please refer to `./app.rb : line 130` and `./lib/route_helpers.rb : line 51` for code implementation

  When an error happens, any valid data that the user has already entered is preserved. Please refer to `./views/add_product.erb : line 5` for one such code example.

- *Validate URL parameters such as ID numbers and query strings.*

  In this project, all `item_id`, `product_id` and `page_num` are validated. Please refer to `./route_helpers.rb : line 67-82` for details.

- *The application must require login authentication.*

  Login authentication is implemented, including following features:
  - Sign-up: `./lib/route_helpers.rb : line 20-31`
  - Sign-in: `./lib/route_helpers.rb : line 7-18`
  - Redirect user to the sign-in page if user is not signed in: `./app.rb : line 33` and `./lib/route_helpers.rb : line 33`
  - Redirect user to the previous URL after sign in: `./lib/route_helpers.rb : line 12 and 35`

## Design Choices

- Sign-in state validation

  I choose to match client's `session_id` with the database's for each request, instead of simply checking whether certain `session` key (for example, `session[:username]`) exists. In my opinion, this choice has three advantages:
  - It is more secure since whoever trying to fake a sign-in state cannot simply create a session key with a random value
  - For security purposes, we don't need to store anything meaningful in the session data, `session_id` is a perfect choice here
  - Being able to control session expiry if neccessary

  However, the trade-off here is one more query to the database for each request, including redirects. For a small scale app, I think this is acceptable.

- Classes and how they interact with different components of the app

  Specifically, I opt to create 3 classes for both `item` and `product` entities. For example, for the `item` entity:
    - `Item` class stores data of a single item, including `product_id`, `best_before_date` etc, with interfaces to read these data
    - `Items` class that stores a collection of `Item` instances. It mixes in the `Enumerable` module and provides various common collection methods: `<`, `each`, `count` etc.
    - `ItemHandler` class that interacts with the database, and create `Items` and `Item` objects so they could be used in Sinatra and `erb` files

  Alternatively, we could also simply creating only one class that interacts with the database, and returns a `PG::Result` object to Sinatra so we could extract data by iterating through it. However, I think former approach is more intuitive and maintainable than dealing with `PG::Result` and `tuple`s, and it has two advantages:

  First, everything stored in a `Tuple` object is `String` objects, which is not always intuitive or even what we want. For example, I may want to count the number of in-stock items and use it to determine how many pages we need. If we rely on the `Tuple` object, we need to remember to call `to_i` somewhere in Sinatra or the `erb` files, even at multiple places. It makes the codebase hard to maintain. By using `ItemHandler` class, we can implement our own interface and make sure the return value is always integer.

  Another advantage I could think of is the code will be more DRY and easier to maintain in case our schema changes.

  However, the trade-off here is code readability, which could be improved by using comments and proper documentation.

## Hope you enjoy the app as much as I enjoy creating it!
