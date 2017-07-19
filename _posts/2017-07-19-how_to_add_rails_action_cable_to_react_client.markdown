---
layout: post
title:  "How to add Rails action cable to React client"
date:   2017-07-19 23:57:45 +0000
---

For this walk through I will assume you have already made the models you would like to connect to action cable.
First thing you will need to do is add action cable to your react app using.

```
npm install actioncable
```
or 
```
yarn add actioncable
```

Next you will need to create the cable setup and export it. I set mine up in its own file.

```
import ActionCable from "actioncable";

const token = localStorage.getItem('token')
const App = {};
App.cable = ActionCable.createConsumer(`ws://localhost:3001/cable?token=${token}`);

export default App.cable

```

The first line imports actioncable so it. Next I grabbed  my JWT token from storage because my app uses JWT for authentication. You can omit this part if you use another method for authentication. The command online five sets up the connection with the URL to my Api. If you aren't using JWT for authentication then you setup should look like this 

```
import ActionCable from "actioncable";

const App = {};
App.cable = ActionCable.createConsumer(`ws://localhost:3001/cable`);

export default App.cable
```

Next we move to the rails api to create our channels. First we will need to un comment out redis from our gem file. Then run bundle install. Then we modify our config/cable.yml to use redis. This part is optional if you don't plan on deploying your app.

```
gem 'redis', '~> 3.2'

bundle install

#config/cable.yml

adapter: redis
url: YOUR_URL
```

Since I am building an instant messaging app I will need a chatrooms channel using the rails generator. You should change chatrooms to the name of the channel that best suits the model that you want to run over actioncable. The output will setup an empty channel that should look like this.

```

rails g channel chatrooms

class ChatroomsChannel < ApplicationCable::Channel
  def subscribed
    # stream_from "some_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
end
```

Before we define any methods here we need to setup our connection.rb file inside channels/action_cable. Inside this file we will tell rails how to authenticate a connection. 

```
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_token_user
      logger.add_tags 'ActionCable', "User #{current_user.id}"
    end
		
  end
end
```

Here on line 3 we are using a rails method called identified_by which is called before a connection is instantiated. next we def connect which is where current_user is defined. As I am using JWT for authentication I passed the token along as a param from my react app. I then authenticate the user using a custom method which we will get to next. On line 7 I added a logger for when a user connects so it will be easier to debug problems.

```
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_token_user
      logger.add_tags 'ActionCable', "User #{current_user.id}"
    end

    protected 
      def find_token_user
        begin
          token = request.params[:token]
          decoded_token = Auth.decode_token(token)
          user_id = decoded_token[0]["user_id"]
          if current_user = User.find_by(id: user_id)
            current_user
          else
            reject_unauthorized_connection
          end
        rescue
          reject_unauthorized_connection
        end
      end
  end
end
```

Next we need to make a private method that is used to authenticate a user. I have my own custom class for debugging the JWT token which can bee seen here.

```
class Auth

  def self.create_token(user_id)
    payload = { user_id: user_id }
    token = JWT.encode(payload, ENV['AUTH_SECRET'], ENV['AUTH_ALGORITHM']) 
  end

  def self.decode_token(token)
    decoded = JWT.decode(token, ENV['AUTH_SECRET'], true, { algorithm: ENV['AUTH_ALGORITHM'] })
  end

end

```

If you are using devise for you user authentication then your authorization method should look like this.

```
def find_verified_user # this checks whether a user is authenticated with devise
      if verified_user = env['warden'].user
        verified_user
      else
        reject_unauthorized_connection
      end
    end
```

If this authentication fails then we use a rails built in method called reject_unauthorized_connection. Next we will mount our cable in our routes. Make sure to use the same path from when you created consumer. This is what my routes looks like, take note of the last line where I mount actioncable. 

```

Rails.application.routes.draw do
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
  namespace :api do
    namespace :v1 do
    	resources :users, only: [:create]
      post '/auth', to: 'auth#login'
      post 'auth/refresh', to: 'auth#refresh'
      post 'chatrooms/direct_message', to: 'chatrooms#direct_message'
      patch 'users', to: 'users#update'
      get 'users/:username', to: 'users#show'
      resources :chatrooms do
        resource :chatroom_users, only: [:create, :destroy]
        resources :messages, only: [:create]
      end
    end
  end 

  mount ActionCable.server => '/cable'
end
```

Next lets define our methods for when a user successfully authenticates and connects.

```
class ChatroomsChannel < ApplicationCable::Channel
  
  def subscribed
    current_user.chatrooms.each do |chatroom|
      stream_from "chatroom: #{chatroom.id}"
    end
  end
  
  def send_message(data)
    @chatroom = Chatroom.find(data["chatroom_id"])
    message = @chatroom.messages.create(body: data["body"], user: current_user) 
    MessageRelayJob.perform_later(message)
  end

  def unsubscribed
    stop_all_streams
  end
end

```

There is a lot going on here so lets break it down. In the subscribe method, when a user connects actioncable will begin streaming from each chatroom they are apart of. Each stream must have a unique name so the chatroom Id is used to ensure that. If you would like to stream from a specific channel then the channel id will be passed as param and your method will look like this. 

```
stream_from "chat_rooms_#{params['chat_room_id']}_channel"
```

---------------------------

```
rails g job "#{YOUR-JOB-NAME-HERE}"
```

Next we define a custom method called send_message which we will call client side to send a message. The param chatroom_id is passed in to identify the chatroom. Then the message is created using the params from the data. The new message is then sent to a background worker job. This is to handle situations where you app can be under alot of load from many users. A worker job can be created by running the following command 

```
class MessageRelayJob < ApplicationJob
  queue_as :default

  def perform(message)
    ActionCable.server.broadcast("chatroom: #{message.chatroom_id}",
      Api::V1::MessagesController.render(message)
    )
  end
end

```

Inside the new job file we create a method called perform which we can call using the following command:

```
NameOfYourJob.perform_later(argument) 
```

Inside our perform method we call ActionCable.server.broadcast(). This takes an argument of the channel name that you are broadcasting to and the data to broadcast. Last but not least we define our unsubscribe method which will is called when a user unsubscribes. 

```
import React from 'react';
import ReactDOM from 'react-dom';
import registerServiceWorker from './registerServiceWorker';
import { Provider } from 'react-redux'
import 'semantic-ui-css/semantic.min.css';
import App from './containers/App';
import store from './redux/store';
import apiCable from './redux/helpers/ApiCable'
import { StyleSheet, css  } from 'aphrodite';

const styles = StyleSheet.create({
    body: {
        backgroundImage: 'linear-gradient( 135deg, #F05F57 0%, #360940 100%);',
        minHeight: '900px'
    }
});

ReactDOM.render(
	<Provider store={store}>
	  <div className={css(styles.body)}>
	  	<App apiCable={apiCable}/>
	  </div>
	</Provider>, 
	document.getElementById('root')
  );

```

Now back to our react client to finish the setup.

```
import React, { Component } from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom'
import NavBar from '../components/NavBar'
import Profile from '../components/Profile'
import Auth from '../components/Auth'
import {ErrorShow} from './ErrorShow'
import {connect} from 'react-redux'
import ChatContainer from './ChatContainer'

const Chats = userIsAuthenticated(ChatContainer)
const NotFound = () =>(<h1>Wrong turn</h1>)
const Home = ()=>(<h1>Welcome To Chit Chat</h1>)
const Pro= userIsAuthenticated(Profile)


class App extends Component {

  componentDidMount() {
    const token = localStorage.getItem('token');
    if (token) {
      this.props.authenticate();
    } else {
      this.props.authenticationFail();
    }
  }
  render() {
    const {errors, apiCable, clearErrors} = this.props
    return (
      <Router>
        <div>
          <div className="navbar">
            <NavBar/>
          </div>
          {errors.length ? <ErrorShow clearErrors={clearErrors}errors={errors}/> : null}
          <Switch>
            <Route exact path="/" component={Home} />
            <Route path="/chats" render={(props)=> <Chats {...props} apiCable={apiCable}/> }/>
            <Route path="/profile/" component={Pro}/>
            <Route component={NotFound}/>
          </Switch>
        </div>
      </Router>
    );
  }
}
```

Here we are importing the apiCable that we created at the very beginning and passing it to our app as a prop. We can then pass this apiCable to any component we would like.  The way I did it was thru the render component for path="/chats":

```
  componentDidMount() {
    this.props.getChats()
    if (!this.props.apiCable.messenger) {
        this.props.apiCable.messenger = this.props.apiCable.subscriptions.create('ChatroomsChannel',{
        connected: function() { console.log("cable: connected") },             // onConnect 
        disconnected: function() { console.log("cable: disconnected") },       // onDisconnect 
        received: (data) => this.props.updateMesages(data)        // OnReceive
      }) 
    } 
  }
	
```
Inside the component that will be communicating over actioncable put the logic for setup inside 'componentDidMount' function. This will ensure that the cable connects right away.

My updateMessages function is taking data from the server and adding it to our redux store. The body of the data sent from our clieant is taken from the form and the chatroom id is taken from the params in the url.  We call perform on our subscription to call the custom method that we defined earlier. This function takes in an argument of the method name and the data. The data and function can be seen below. 
```

 handleSubmit=(body)=>{
    const message = {chatroom_id: this.props.match.params.chatId, body: body}
    this.props.apiCable.messenger.perform("send_message", message)
  }


const recieveMessage = (message) =>{
	return {
		type: 'ADD_MESSAGE',
	  message
	}
}

export const updateMesages = (message) =>{
  return dispatch =>{
    var data = JSON.parse(message)
  	dispatch(recieveMessage(data))
  }
}
	
```

Our updateMessages function parses the data because our server does not send back JS objects. Once these datat from actioncable is added to the redux store it can then be rendered to the page.

Hopefully, you found this article useful and interesting.





