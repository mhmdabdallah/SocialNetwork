#SocialNetwrok
class SocialNetwork:
  def __init__(self):
      self.graph = {}  
      self.friend_requests = {}  
      self.user_profiles = {}  
      self.messages = {}  
      self.tweets = {} 

  def add_user(self):
      print("\nCreate an account:")
      user_id = input("Username: ")
      password = input("Password: ")
      hobbies = input("Hobbies (comma-separated): ").split(',')
      hobbies = [hobby.strip() for hobby in hobbies]
      interests = input("Interests (comma-separated): ").split(',')
      interests = [interest.strip() for interest in interests]

      if user_id.lower() not in [uid.lower() for uid in self.graph]:
          self.graph[user_id] = []  
          self.friend_requests[user_id] = []  
          self.user_profiles[user_id] = {"password": password, "hobbies": hobbies, "interests": interests} 
          self.messages[user_id] = []
          print(f"Account created successfully! Welcome, {user_id}!")
      else:
          print(f"Account with username {user_id} already exists!")

  def login(self):
      print("\nLogin:")
      user_id = input("Username: ")
      password = input("Password: ")

      for uid in self.user_profiles:
          if uid.lower() == user_id.lower() and self.user_profiles[uid]["password"] == password:
              print(f"Login successful! Welcome, {uid}!")
              self.home(uid)
              return

      print("Invalid username or password!")

  def add_friend_request(self, user_id):
      while True:
          print("\nSend a friend request:")
          to_user = input("Enter the username of the user you want to send a friend request to (or 'back' to go back): ")

          if to_user.lower() == 'back':
              return

          for uid in self.graph:
              if uid.lower() == to_user.lower():
                  to_user = uid
                  break
          else:
              print(f"User {to_user} does not exist!")
              continue

          if to_user not in self.friend_requests[user_id] and user_id not in self.graph[to_user]:
              self.friend_requests[to_user].append(user_id)
              print(f"Friend request sent from {user_id} to {to_user} successfully!")
              return
          else:
              print(f"You have already sent a friend request to {to_user} or are already friends!")
              return

  def accept_friend_request(self, user_id):
      while True:
          print("\nAccept a friend request:")
          from_user = input("Enter the username of the user who sent you a friend request (or 'back' to go back): ")

          if from_user.lower() == 'back':
              return

          for uid in self.friend_requests[user_id]:
              if uid.lower() == from_user.lower():
                  from_user = uid
                  break
          else:
              print(f"No pending friend request from {from_user}!")
              continue

          self.graph[user_id].append(from_user) 
          self.graph[from_user].append(user_id)  
          self.friend_requests[user_id].remove(from_user)  
          print(f"Friend request from {from_user} accepted successfully!")
          return

  def remove_friend(self, user_id):
      while True:
          print("\nRemove a friend:")
          friend_to_remove = input("Enter the username of the friend you want to remove (or 'back' to go back): ")

          if friend_to_remove.lower() == 'back':
              return

          if friend_to_remove in self.graph[user_id]:
              self.graph[user_id].remove(friend_to_remove)  
              self.graph[friend_to_remove].remove(user_id)  
              print(f"Friend {friend_to_remove} removed successfully!")
              return
          else:
              print(f"You are not friends with {friend_to_remove}!")
              return

  def see_friends(self, user_id):
      while True:
          print("\nYour friends:")
          for friend in self.graph[user_id]:
              print(friend)
          print("\nEnter 'back' to go back:")
          choice = input("Enter your choice: ")
          if choice.lower() == 'back':
              return

  def see_all_users(self):
      while True:
          print("\nAll users:")
          for user_id, profile in self.user_profiles.items():
              print(f"Username: {user_id}")
              print(f"Hobbies: {', '.join(profile['hobbies'])}")
              print(f"Interests: {', '.join(profile['interests'])}")
              print()
          print("\nEnter 'back' to go back:")
          choice = input("Enter your choice: ")
          if choice.lower() == 'back':
              return

  def recommend_friends(self, user_id):
      while True:
          print("\nGet friend recommendations:")
          user_profile = self.user_profiles[user_id]
          hobbies = user_profile["hobbies"]
          interests = user_profile["interests"]

          similarity_scores = {}
          for other_user in self.graph:
              if other_user!= user_id:
                 other_profile = self.user_profiles[other_user]
              hobby_similarity = len(set(hobbies) & set(other_profile["hobbies"])) / len(set(hobbies) | set(other_profile["hobbies"]))
              interest_similarity = len(set(interests) & set(other_profile["interests"])) / len(set(interests) | set(other_profile["interests"]))
              similarity_score = hobby_similarity + interest_similarity
              similarity_scores[other_user] = similarity_score

          recommended_friends = []
          for other_user in sorted(similarity_scores, key=similarity_scores.get, reverse=True)[:5]:
              other_profile= self.user_profiles[other_user]
              common_hobbies = set(hobbies) & set(other_profile["hobbies"])
              common_interests = set(interests) & set(other_profile["interests"])
              explanation = f"Shared hobbies: {', '.join(common_hobbies)}; Shared interests: {', '.join(common_interests)}"
              recommended_friends.append((other_user, explanation))

          print("Recommended friends:")
          for friend in recommended_friends:
              print(f"{friend[0]} - {friend[1]}")
          print("\nEnter 'back' to go back:")
          choice = input("Enter your choice: ")
          if choice.lower() == 'back':
              return

  def send_message(self, user_id):
      while True:
          print("\nSend a message:")
          to_user = input("Enter the username of the user you want to send a message to (or 'back' to go back): ")

          if to_user.lower() == 'back':
              return

          for uid in self.graph:
              if uid.lower() == to_user.lower():
                  to_user = uid
                  break
          else:
              print(f"User {to_user} does not exist or you are not friends!")
              continue

          message = input("Enter your message: ")

          if to_user in self.graph[user_id]:
              if user_id not in self.messages:
                  self.messages[to_user]=[]
              self.messages[to_user].append(f"From {user_id}: {message}")
              print(f"Message sent to {to_user} successfully!"}
              return
          else:
              print(f"You are not friends with {to_user}")
              return

  def see_messages(self, user_id):
      while True:
          print("\nYour messages:")
          if user_id in self.messages:
              for message in self.messages[user_id]:
                  print(message)
          else:
              print("No messages!")
          print("\nEnter 'back' to go back:")
          choice = input("Enter your choice: ")
          if choice.lower() == 'back':
              return

  def post_tweet(self, user_id):
        while True:
            print("\nPost a tweet:")
            tweet = input("Enter your tweet (or 'back' to go back): ")

            if tweet.lower() == 'back':
                return

            if user_id not in self.tweets:
                self.tweets[user_id] = []
            self.tweets[user_id].append(tweet)
            print(f"Tweet posted successfully!")
            return

  def view_tweets(self, user_id):
        while True:
            print("\nView tweets:")
            for user, tweets in self.tweets.items():
                for tweet in tweets:
                    print(f"{user}: {tweet}")
            print("\nEnter 'back' to go back:")
            choice = input("Enter your choice: ")
            if choice.lower() == 'back':
                return


  def home(self, user_id):
      while True:
          print("\nHome")
          print("1.Send a friend request")
          print("2. Accept a friend request")
          print("3. See friends")
          print("4. See all users")
          print("5. Get friend recommendations")
          print("6. Remove a friend")
          print("7. Send a message")
          print("8. See messages")
          print("9. Post a tweet")
          print("10. View tweets")
          print("11. Logout")
          choice = input("Enter your choice: ")

          if choice == "1":
              self.add_friend_request(user_id)
          elif choice == "2":
              self.accept_friend_request(user_id)
          elif choice == "3":
              self.see_friends(user_id)
          elif choice == "4":
              self.see_all_users()
          elif choice == "5":
              self.recommend_friends(user_id)
          elif choice == "6":
              self.remove_friend(user_id)
          elif choice == "7":
              self.send_message(user_id)
          elif choice == "8":
              self.see_messages(user_id)
          elif choice == "9":
              self.post_tweet(user_id)
          elif choice == "10":
              self.view_tweets(user_id)
          elif choice == "11":
              break
          else:
              print("Invalid choice!")

  def run(self):
      while True:
          print("\nSocial Network App")
          print("1. Create an account")
          print("2. Login")
          print("3. Exit")
          choice = input("Enter your choice: ")

          if choice == "1":
              self.add_user()
          elif choice == "2":
              self.login()
          elif choice == "3":
              break
          else:
              print("Invalid choice!")

if __name__ == "__main__":
  sn = SocialNetwork()
  sn.run()