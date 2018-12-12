# Optimizely Ruby Full Stack Tutorial

This tutorial enables you to quickly get started in your development efforts to create a Ruby full-stack web application with the Optimizely X Web SDK. For general information Optimizely X for Ruby, see the [documentation](https://developers.optimizely.com/x/solutions/sdks/introduction/index.html?language=ruby).

![public/images/screenshot.png](public/images/screenshot.png)

The Test App works as follows:

* The loaded application initializes the Optimizely manager starting the datafile fetch.
* Accessing the `index` page populates and displays the catalog item list.
* Applying a filter posts data to the `shop` page and sorts the catalog items through `optimizely.activate()`. The application then navigates to the `index` page.
* Clicking on a **Buy Now** button tracks the purchase using `optimizely.track()` and sends a conversion event for the event named `purchased_item`. The application then navigates to the `buy` page.

## Prerequisites

* Ruby with Gems
* GitHub account configured with [SSH keys](https://help.github.com/articles/connecting-to-github-with-ssh/)

## Quick start

This section shows you how to prepare, build, and run the sample application using the command line.

This section provides the steps to build the project and execute its various tests from the command line, and includes commands to discover additional tasks.

1. Open a terminal window.

2. Navigate to the project folder:
```shell
cd /path/to/project
```

2. Install the Optimizely SDK:
```shell
gem install optimizely-sdk
```

3. Install three Ruby dependencies:

	- Install `httparty`: 
``` shell
sudo gem install -n /usr/local/bin/ httparty
```
	- Install `sinatra`: 
``` shell
sudo gem install -n /usr/local/bin/ sinatra
```
	- Install `csv_hasher`: 
``` shell
sudo gem install -n /usr/local/bin/ csv_hasher
```

4. Start the app server:
```shell
ruby bin/app.rb 
```

5. Navigate to to [http://localhost:8080](http://localhost:8080) in your browser.

**Note:**

- You can update the Optimizely variation code in the [bin/app.rb](bin/app.rb) file:
```ruby
post '/shop' do
  ...

  variation_key = optimizely.activate('feature_rollout', user_id)
  if variation_key == 'holdback'
    items = items.sort_by { |k| k['category'].to_i }
  elsif variation_key == 'new_feature'
    # execute code for new_feature
    feature_display = 'block'
    items = items.sort_by { |k| k[filter_by].to_i }
  else
    # execute default code
    items = items.sort_by { |k| k['category'].to_i }
  end

  ...
end
```
- The app depends on third-party modules. Changes to any source code within the modules will be applied to the app during the next build.

## How the Test App was Created

The following subsections provide information about key aspects of the Test App and how it was put together:

* [Dependencies](#dependencies)
* [User Interface and Visual Assets](#user-interface-and-visual-assets)
* [Create the Main Application File](#create-the-main-application-file)

### Dependencies

This project has four dependencies: 

1. **Optimizely SDK**: contains the Optimizely X Web SDK with the following two primary responsibilities:
 * Handles downloading the Optimizely datafile and building Optimizely objects.
 * Delivers the compiled Optimizely object to listeners and caches it in memory.

2. **HTTParty**: exposes all of the standard HTTP request methods, like GET, POST, PUT, and DELETE.

3. **Sinatra**: base framework for the web application.

4. **CSV_hasher**: converts a specified CSV file into array of hashes.

For details about the APIs used to develop this sample, see the [documentation](https://docs.developers.optimizely.com/full-stack/docs).


### User Interface and Visual Assets

The following layout files are in the **/views** directory:

Asset|Description
----|----
`index.erb`|Displayed when the application loads.
`buy.erb`|Displays when a purchase event occurs.


The following art files in the **/public/images** directory are used as background images for the various activities:

Asset|Description
----|----
`logo.png`|Contains the logo image for the app.
`screenshot.png`|Contains a screenshot of the rendered catalog.
`item_1.png`|Contains the product image for the `Derby Hat` catalog item.
`item_2.png`|Contains the product image for the `Bo Henry` catalog item.
`item_3.png`|Contains the product image for the `The Go Bag` catalog item.
`item_4.png`|Contains the product image for the `Springtime` catalog item.
`item_5.png`|Contains the product image for the `The Night Out` catalog item.
`item_6.png`|Contains the product image for the `Dawson Trolley` catalog item.
`item_7.png`|Contains the product image for the `Long Sleeve Swing Shirt` catalog item.
`item_8.png`|Contains the product image for the `Long Sleever Tee` catalog item.
`item_9.png`|Contains the product image for the `Simple Cardigan` catalog item.

### Create the Main Application File

The code samples in this section are in the [**bin/app.rb**](bin/app.rb) file.

Connect the required dependencies to the application, including the Optimizely SDK using `require`.

```ruby
require 'sinatra'
require 'httparty'
require 'optimizely'
require 'csv_hasher'
```

Initialize Optimizely. 

- Define the `url` for the Optimizely data and retrieve the information using `HTTParty.get()`.
- Initialize a new `Optimizely` object with the data using `Optimizely::Project.new()`.

```ruby
url = 'https://cdn.optimizely.com/json/7681190327.json'
datafile = HTTParty.get(url).body
optimizely = Optimizely::Project.new(datafile)
```

Define the app settings and file references:

Setting|Value|Description
---|---|---
`port`|`8080`|Port for the URL address
`static`|`true`|Indicates assets are static
`public_folder`|`public`|Asset location 
`views`|`views`|View location

```ruby
set :port, 8080
set :static, true
set :public_folder, "public"
set :views, "views"
```

Define the structure for the `Item` class:

Property|Description
---|---
`name`|Name of the catalog item
`color`|Color of the catalog item
`category`|Category where the catalog item belongs
`price`|Price for the catalog item
`imageUrl`|Image reference for the catalog item

```ruby
class Item <
  Struct.new(:name, :color, :category, :price, :imageUrl)
end
```

The `build_items()` method generates an array of catalog items.

1. Initialize a new `Array`.
2. Open the `bin/items.csv` file
3. For each line item in the CSV file:
	- Define a new `Item` object.
	- Set the appropriate properties for `item`.
	- Add the `item` to the `items` array.
4. return the completed `items` array.

```ruby
def build_items()
  items = Array.new
  f = File.open("bin/items.csv", "r")
  f.each_line { |line|

    fields = line.split(',')

    item = Item.new

    item.name = fields[0]
    item.color = fields[1]
    item.category = fields[2]
    item.price = fields[3]
    item.imageUrl = fields[4]
    items.push(item)
  }
  return items
end
```

When the root URL is accessed:

1. Build the items array using `build_items()`.
2. Load the `index` layout with the following properties:

Property|Value|Description
---|---|---
`user_id`|empty string|User ID
`filter_by`|empty string|Filtering parameters
`feature_display`|`none`|Feature display format is off
`data`|`items`|Catalog data

```ruby
get '/' do
  items = build_items()

  erb :index, :locals => {'user_id' => '', 'filter_by' => '', 'feature_display' => 'none', 'data' => items}

end
```

When data is posted to the `/shop` URL:

1. Set the  `user_id` and `filter_by` variables with the posted data, and set `feature_display` to `none`.
2. Build the items array using `build_items()`.
3. Apply `user_id` to `feature_rollout` in the Optimizely SDK using `optimizely.activate()`.
4. Sort `items` based on the `variation_key` using `items.sort_by` and set `items` to this sorted value.
5. Load the `index` layout with the new properties.

```ruby
post '/shop' do
  user_id = params[:user_id]
  filter_by = params[:filter]
  feature_display = 'none'
  items = build_items()

  variation_key = optimizely.activate('feature_rollout', user_id)
  if variation_key == 'holdback'
    items = items.sort_by { |k| k['category'].to_i }
  elsif variation_key == 'new_feature'
    # execute code for new_feature
    feature_display = 'block'
    items = items.sort_by { |k| k[filter_by].to_i }
  else
    # execute default code
    items = items.sort_by { |k| k['category'].to_i }
  end

  erb :index, :locals => {'user_id' => user_id, 'filter_by' => filter_by, 'feature_display' => feature_display, 'data' => items}
end
```

When data is posted to the `/buy` URL:

1. Set the  `user_id` variable with the posted data.
2. Track the purchased item by applying `user_id` to `purchased_item` in the Optimizely SDK using `optimizely.track()`.
3. Load the `buy` layout.

```ruby
post '/buy' do
  user_id = params[:user_id]
  optimizely.track('purchased_item', user_id)
  erb :buy, :locals => {}
end
```