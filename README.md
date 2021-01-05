# README

Rails 6.1 ActionCable

## Create rails app:

```
rails new cableguy --database=mysql

cd cableguy
```

## Create a model

```
rails g resource libraryBook title status status_date:datetime

rails db:create
rails db:migrate
```

## Add seeds:

```
libary_books = LibraryBook.create([{title: 'Moby Dick', status: 'on-shelf'}, {title: 'Cruel Shoes', status: 'on-shelf'}])

rails db:seed
```

## Update routes.rb
```
root "library_books#index"
```

## Get all the books in the controller
```
library_books_controller.rb
def index
    @books = LibraryBook.all
end
```
## create index in app/views/libary_books/index.html.erb
```
<h1>Our Books</h1>

<% if !@books.blank? %>
  <% for book in @books %>
    
  <% end %>
<% end %>
```
## Add bootstrap to layout application.html.erb
```
    <script src="https://code.jquery.com/jquery-3.5.1.min.js" integrity="sha256-9/aliU8dGd2tb6OSsuzixeV4y/faTqgFtohetphbbj0=" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.bundle.min.js" integrity="sha384-LtrjvnR4Twt/qOuYxE721u19sVFLVSA4hf/rRt6PrZTmiPltdZcI7q7PXQBYTKyf" crossorigin="anonymous"></script>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css" integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
```

## wrap yield in container div
```
    <div class='container'>
    <%= yield %>
    </div>
```

## quick check of books index.html.erb:
```
    <div class='row'>
        <div class='col'>
        <%= book.title%>
        </div>
        <div class='col'>
        <%= book.status.titleize%>
        </div>
    </div>
```

### create a partial to render a book
### move code into it
```
app/views/library_books/_one_book.html.erb
    <div class='row'>
        <div class='col'>
        <%= book.title%>
        </div>
        <div class='col'>
        <%= book.status.titleize%>
        </div>
    </div>
```

## call partial in index.html.erb
```
  <%= render partial: "one_book", locals: {book: book} %>
```

######## end of the basics ########

# add stimulus
```
yarn add stimulus
```

# add stimulus stuff to application.js
```
import { Application } from "stimulus"
import { definitionsFromContext } from "stimulus/webpack-helpers"

const application = Application.start()
const context = require.context("../controllers", true, /\.js$/)
application.load(definitionsFromContext(context))
```
# create a stimulus controller app/javascript/contollers/book_status_controller.js
```
import { Controller } from "stimulus"
import consumer from '../channels/consumer';

export default class extends Controller {
  static targets = ['bookstatus']

  connect() {
    console.log('Will create subscription to: channel: "LibraryBookStatusChannel" library_book_id: ' + this.data.get('bookid'));

    this.channel = consumer.subscriptions.create({ channel: 'LibraryBookStatusChannel', library_book_id: this.data.get('bookid') }, {
      connected: this._cableConnected.bind(this),
      disconnected: this._cableDisconnected.bind(this),
      received: this._cableReceived.bind(this),
    });
  }

  _cableConnected() {
    // Called when the subscription is ready for use on the server
    console.log('_cableConnected');
  }

  _cableDisconnected() {
    // Called when the subscription has been terminated by the server
    console.log('_cableDisconnected');
  }

  _cableReceived(data) {
    console.log('_cableReceived');
    // Called when there's incoming data on the websocket for this channel
    this.bookstatusTarget.innerHTML = data.message;
  }
}
```

# add stimulus stuff to index.html.erb
```
    <div data-controller="book-status" data-book-status-bookid="<%= book.id %>">
      <div data-book-status-target='bookstatus'>
        <%= render partial: "one_book", locals: {book: book} %>
      </div>
    </div>
```

# restart server

# see output in chrome console

# see nothing gets update even we update model

#### model stuff ####

# create channel in app/channels/library_book_status_channel.rb
```
class LibraryBookStatusChannel < ApplicationCable::Channel
  def subscribed
  # the param name is set in the stimulus controller
    stream_from "LibraryBookStatusChannel:#{params[:library_book_id]}"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
    stop_all_streams
  end
end
```
# add callback on after_commit in libary_book.rb
```
  after_commit :broadcast_me

  def broadcast_me
    ActionCable.server.broadcast "LibraryBookStatusChannel:#{id}", {
      status: status.titleize,
      message: LibraryBooksController.render(partial: 'one_book', locals: { book: self }).squish
    }
  end
```

#### why nothing works!?

# add redis
```
in Gemfile
gem 'redis'
```

# update cable.yml
```
development:
    adapter: redis
```
# add action meta tag for production to top of application.html.erb
```
    <%= action_cable_meta_tag %>
```
