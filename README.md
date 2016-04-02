### Integrating with Slack

Now we want our todo app to update slack when a todo item is created and when a todo item is updated, to do this we need the "slack-notifier" gem. We'll also need a slack team account setup and a slack webhook integration setup. You can follow this link to set that up if you alredy have a team: [Slack Integration Setup](https://api.slack.com/incoming-webhooks). You'll get an api url to send data to that will feed into a channel that you selected for your team. 

```ruby
# Gemfile

gem "slack-notifier"
```

```sh
$ bundle install
```

###### We run $ bundle install

###### So what we want to do now is tell rails to send a notification to slack whenever a todo item is created
######    at this point i expect you to have your webhook url from slack, if you haven't please go a step back

###### So we add

```
require 'slack-notifier'
# respond with a success notification and the user project object
notifier = Slack::Notifier.new "https://hooks.slack.com/services/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"
notifier.ping "Todo Added:"+@todo.item, icon_url: "https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcR6t41ErQxx0y1rApv207bM3LznQVdvOILrYy-XTUVg3JpxGvRn"



```

to our # app/controllers/todos_controller.rb create method and we have

app/controllers/todos_controller.rb

```
class TodosController < ApplicationController
  respond_to :json

  before_filter :find_todo, :only => [:show]

  def index
      @todos = Todo.all
      respond_with(@todos)
  end

  def new
    @todo = Todo.new
  end

  def create
    @todo = Todo.create(todo_params)
    if @todo.save
    require 'slack-notifier'
    # respond with a success notification and the user project object
    notifier = Slack::Notifier.new "https://hooks.slack.com/services/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"
    notifier.ping "Todo Added:"+@todo.item, icon_url: "https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcR6t41ErQxx0y1rApv207bM3LznQVdvOILrYy-XTUVg3JpxGvRn"
    respond_with(@todo)
    else
      respond_with(nil, @message = "Error while creating Todo")
    end
  end

  def show
    respond_with(@todo)
  end

  def destroy
    @todo = Todo.find(params[:id])
    if @todo.update(checked: true)
        require 'slack-notifier'
        # respond with a success notification and the user project object
        notifier = Slack::Notifier.new "https://hooks.slack.com/services/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"
        notifier.ping "Todo Completed:"+@todo.item, icon_url: "https://encrypted-tbn3.gstatic.com/images?q=tbn:ANd9GcR6t41ErQxx0y1rApv207bM3LznQVdvOILrYy-XTUVg3JpxGvRn"
        respond_with(@todo)
    else
        respond_with(nil, @message = "Todo Update Failed")
    end
  end


  private
  def todo_params
    params.require(:todo).permit(
        :item,
        :checked,
        :description
    )
  end

  private
  def find_todo
    @todo = Todo.find(params[:id])
  end

end


```
