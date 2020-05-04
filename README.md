# ruby_blog
Basic app in ruby.

### checking ruby version
```
$ ruby -v
$ sqlite3 --version
$ gem install rails
$ rails --version
```
### creating new application
```
$ rails new blog
$ cd blog
```

### installing dependencies
```
$ bundle install
```

### starting up the Web Server
```
$ rails server
```

the go to: http://localhost:3000/

## to see "Hello", you need to create at minimum a controller and a view.

```
$ rails generate controller Welcome index
```

In `app/views/welcome/index.html.erb` replace all the code with:

```
<h1>Hello!</h1>
```

Open the file `config/routes.rb` in your editor and replace the code with:
```
Rails.application.routes.draw do
  get 'welcome/index'
 
  resources :articles

  root 'welcome#index'
end
```

### to check routes
```
$ rails routes
```

## create Article conroller and model to make it work: http://localhost:3000/articles/new
```
$ rails generate controller Articles
```

And in this contoroller add to be alble to create mew Articles:
```
class ArticlesController < ApplicationController
  def new
  end
end
```

And in `app/views/articles/new.html.erb`:
```
<h1>New Article</h1>
```

To change it to form in `app/views/articles/new.html.erb` add:
```
<%= form_with scope: :article, local: true do |form| %>
  <p>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </p>
 
  <p>
    <%= form.label :text %><br>
    <%= form.text_area :text %>
  </p>
 
  <p>
    <%= form.submit %>
  </p>
<% end %>
``` 

And to be able to create Articles we need method `create` in controller:
```

class ArticlesController < ApplicationController
  def new
  end

  def create
    # to check what is in params
    # render plain: params[:article].inspect 
    @article = Article.new(article_params)
    @article.save
    redirect_to @article
  end

  private
  def article_params
    params.require(:article).permit(:title, :text)
  end
end
```

## adding Arfticle model
```
$ rails generate model Article title:string text:text
```

## running a Migration
```
$ rails db:migrate
```

to provide migrations to production:
```
$ rails db:migrate RAILS_ENV=production
```

## add show method to controller

add to ArticlesController:
```
class ArticlesController < ApplicationController
  def show
    @article = Article.find(params[:id])
  end
 
  def new
  end

  def create
      # to check what is in params
      # render plain: params[:article].inspect 
      @article = Article.new(article_params)
      @article.save
      redirect_to @article
  end

  private
  def article_params
      params.require(:article).permit(:title, :text)
  end
end

```


add to `app/views/articles/show.html.erb`:
```
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
 
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
```

## to show all articles add `index` method to controller 
```
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def new
  end

  def show
      @article = Article.find(params[:id])
  end

  def create
      @article = Article.new(article_params)
      @article.save
      redirect_to @article
  end

  private
  def article_params
      params.require(:article).permit(:title, :text)
  end
end
```

and to view `app/views/articles/index.html.erb`:
```
<h1>Listing Articles</h1>
 
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th></th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
    </tr>
  <% end %>
</table>
```

## modifications in main page

add to `app/views/welcome/index.html.erb`:
```
<h1>Hello, Rails!</h1>
<%= link_to 'My Blog', controller: 'articles' %>
``` 

add to `app/views/articles/index.html.erb`:
```
<%= link_to 'New article', new_article_path %>
```

add to `app/views/articles/new.html.erb`:
```
<%= link_to 'Back', articles_path %>
```
add tp `app/views/articles/show.html.erb`:
```
<%= link_to 'Back', articles_path %>
```

## add model to `app/models/article.rb`:
```
class Article < ApplicationRecord
  validates :title, presence: true,
                    length: { minimum: 5 }
end
```

add to `app/views/articles/new.html.erb`:
```
<%= form_with scope: :article, url: articles_path, local: true do |form| %>
 
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
 
  <p>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </p>
 
  <p>
    <%= form.label :text %><br>
    <%= form.text_area :text %>
  </p>
 
  <p>
    <%= form.submit %>
  </p>
 
<% end %>
 
<%= link_to 'Back', articles_path %>
```

## updating Articles 
```
class ArticlesController < ApplicationController
    def index
      @articles = Article.all
    end

    def new
      @article = Article.new
    end

    def show
      @article = Article.find(params[:id])
    end

    def create
      @article = Article.new(article_params)

      if @article.save
        redirect_to @article
      else
        render 'new'
      end
    end

    def edit
      @article = Article.find(params[:id])
    end

    def update
      @article = Article.find(params[:id])
     
      if @article.update(article_params)
        redirect_to @article
      else
        render 'edit'
      end
    end

    private
    def article_params
        params.require(:article).permit(:title, :text)
    end
end
```

add to `app/views/articles/edit.html.erb`:
```
<h1>Edit Article</h1>
 
<%= form_with(model: @article, local: true) do |form| %>
 
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
 
  <p>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </p>
 
  <p>
    <%= form.label :text %><br>
    <%= form.text_area :text %>
  </p>
 
  <p>
    <%= form.submit %>
  </p>
 
<% end %>
 
<%= link_to 'Back', articles_path %>
```

add link to `app/views/articles/index.html.erb`:
```
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th colspan="2"></th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
      <td><%= link_to 'Edit', edit_article_path(article) %></td>
    </tr>
  <% end %>
</table>
```
add to `app/views/articles/show.html.erb`:
```
<%= link_to 'Edit', edit_article_path(@article) %> |
<%= link_to 'Back', articles_path %>
```

## add to `app/views/articles/_form.html.erb`:
```
<%= form_with model: @article, local: true do |form| %>
 
  <% if @article.errors.any? %>
    <div id="error_explanation">
      <h2>
        <%= pluralize(@article.errors.count, "error") %> prohibited
        this article from being saved:
      </h2>
      <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
 
  <p>
    <%= form.label :title %><br>
    <%= form.text_field :title %>
  </p>
 
  <p>
    <%= form.label :text %><br>
    <%= form.text_area :text %>
  </p>
 
  <p>
    <%= form.submit %>
  </p>
 
<% end %>
```

add to `app/views/articles/new.html.erb`:
```
<h1>New Article</h1>
 
<%= render 'form' %>
 
<%= link_to 'Back', articles_path %>
```

add to `app/views/articles/edit.html.erb`:
```
<h1>Edit Article</h1>
 
<%= render 'form' %>
 
<%= link_to 'Back', articles_path %>
```

## delete article

add to controller
```
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def new
    @article = Article.new
  end

  def show
    @article = Article.find(params[:id])
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render 'new'
    end
  end

  def edit
    @article = Article.find(params[:id])
  end

  def update
    @article = Article.find(params[:id])
    
    if @article.update(article_params)
      redirect_to @article
    else
      render 'edit'
    end
  end

  def destroy
    @article = Article.find(params[:id])
    @article.destroy
  
    redirect_to articles_path
  end

  private
  def article_params
      params.require(:article).permit(:title, :text)
  end
end

```

add to `app/views/articles/index.html.erb`:
```
<h1>Listing Articles</h1>
<%= link_to 'New article', new_article_path %>
<table>
  <tr>
    <th>Title</th>
    <th>Text</th>
    <th colspan="3"></th>
  </tr>
 
  <% @articles.each do |article| %>
    <tr>
      <td><%= article.title %></td>
      <td><%= article.text %></td>
      <td><%= link_to 'Show', article_path(article) %></td>
      <td><%= link_to 'Edit', edit_article_path(article) %></td>
      <td><%= link_to 'Destroy', article_path(article),
              method: :delete,
              data: { confirm: 'Are you sure?' } %></td>
    </tr>
  <% end %>
</table>
```

## add Comment 
```
$ rails generate model Comment commenter:string body:text article:references
$ rails db:migrate
```

add to `app/models/article.rb`:
```
class Article < ApplicationRecord
  has_many :comments
  validates :title, presence: true,
                    length: { minimum: 5 }
  end
```

add to `config/routes.rb`:
```
resources :articles do
  resources :comments
end
```

## add Comment controller
```
$ rails generate controller Comments
```

add to `app/views/articles/show.html.erb`:
```
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
 
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
 
<h2>Add a comment:</h2>
<%= form_with(model: [ @article, @article.comments.build ], local: true) do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>
 
<%= link_to 'Edit', edit_article_path(@article) %> |
<%= link_to 'Back', articles_path %>
```

add to `app/controllers/comments_controller.rb`:
```
class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end
 
  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end
```

add to `app/views/articles/show.html.erb`:
```
<p>
  <strong>Title:</strong>
  <%= @article.title %>
</p>
 
<p>
  <strong>Text:</strong>
  <%= @article.text %>
</p>
 
<h2>Comments</h2>
<% @article.comments.each do |comment| %>
  <p>
    <strong>Commenter:</strong>
    <%= comment.commenter %>
  </p>
 
  <p>
    <strong>Comment:</strong>
    <%= comment.body %>
  </p>
<% end %>
 
<h2>Add a comment:</h2>
<%= form_with(model: [ @article, @article.comments.build ], local: true) do |form| %>
  <p>
    <%= form.label :commenter %><br>
    <%= form.text_field :commenter %>
  </p>
  <p>
    <%= form.label :body %><br>
    <%= form.text_area :body %>
  </p>
  <p>
    <%= form.submit %>
  </p>
<% end %>
 
<%= link_to 'Edit', edit_article_path(@article) %> |
<%= link_to 'Back', articles_path %>
```

