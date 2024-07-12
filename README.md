#SocialNetwrok
import json
import networkx as nx


class SocialNetwork:

    def __init__(self):#O(1)
        self.load_data()

    def load_data(self):#O(1)
        try:
            with open('data.json', 'r') as f:
                data = json.load(f)
                self.graph = data.get('graph', {})
                self.friend_requests = data.get('friend_requests', {})
                self.user_profiles = data.get('user_profiles', {})
                self.messages = data.get('messages', {})
                self.tweets = data.get('tweets', {})
                self.likes = data.get('likes', {})
        except FileNotFoundError:
            with open('data.json', 'w') as f:
                json.dump({}, f)
            self.graph = {}
            self.friend_requests = {}
            self.user_profiles = {}
            self.messages = {}
            self.tweets = {}
            self.likes = {}
        except (FileNotFoundError, json.JSONDecodeError, IOError) as e:
            print(f"Error loading data: {e}")
            self.graph = {}
            self.friend_requests = {}
            self.user_profiles = {}
            self.messages = {}
            self.tweets = {}
            self.likes = {}

    def save_data(self):#O(1)
        data = {
            'graph': self.graph,
            'friend_requests': self.friend_requests,
            'user_profiles': self.user_profiles,
            'messages': self.messages,
            'tweets': self.tweets,
            'likes': self.likes,
        }
        try:
            with open('data.json', 'w') as f:
                json.dump(data, f, indent=4)
        except IOError as e:
            print(f"Error saving data: {e}")

    def add_user(self):#O(n) it checks if the user_id exist in the graph where we have n users
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
            self.user_profiles[user_id] = {
                "password": password,
                "hobbies": hobbies,
                "interests": interests
            }
            self.messages[user_id] = []
            self.save_data()
            print(f"Account created successfully! Welcome, {user_id}!")
        else:
            print(f"Account with username {user_id} already exists!")

    def login(self):#O(n) it try all the user to find a match
        print("\nLogin:")
        user_id = input("Username: ")
        password = input("Password: ")

        for uid in self.user_profiles:
            if uid.lower() == user_id.lower(
            ) and self.user_profiles[uid]["password"] == password:
                print(f"Login successful! Welcome, {uid}!")
                self.home(uid)
                return

        print("Invalid username or password!")

    def add_friend_request(self, user_id):#O(1)
        while True:
            print("\nSend a friend request:")
            to_user = input(
                "Enter the username of the user you want to send a friend request to (or 'back' to go back): "
            )

            if to_user.lower() == 'back':
                return

            for uid in self.graph:
                if uid.lower() == to_user.lower():
                    to_user = uid
                    break
            else:
                print(f"User {to_user} does not exist!")
                continue

            if to_user not in self.friend_requests[
                    user_id] and user_id not in self.graph[to_user]:
                self.friend_requests[to_user].append(user_id)
                print(
                    f"Friend request sent from {user_id} to {to_user} successfully!"
                )
                self.save_data()
                return
            else:
                print(
                    f"You have already sent a friend request to {to_user} or are already friends!"
                )
                return

    def accept_friend_request(self, user_id):#O(n) iterates over all friend requests which have n requests
        while True:
            print("\nAccept a friend request:")
            if user_id in self.friend_requests and self.friend_requests[
                    user_id]:
                print("Friend requests:")
                for i, from_user in enumerate(self.friend_requests[user_id]):
                    print(f"{i+1}.{from_user}")
                choice = input(
                    "Enter the number of the friend request you want to accept (or 'back' to fo back): "
                )
                if choice.lower() == 'back':
                    return
                try:
                    choice = int(choice)
                    from_user = self.friend_requests[user_id][choice - 1]
                    self.graph[user_id].append(from_user)
                    self.graph[from_user].append(user_id)
                    self.friend_requests[user_id].remove(from_user)
                    print(
                        f"Friend request from {from_user} accepted successfully!"
                    )
                    self.save_data()
                    return
                except (ValueError, IndexError):
                    print("invalid choice!")

            else:
                print("No friend Requests!")
                return

    def remove_friend(self, user_id):#O(1) checks if friend exist in the graph, constante time
        while True:
            print("\nRemove a friend:")
            friend_to_remove = input(
                "Enter the username of the friend you want to remove (or 'back' to go back): "
            )

            if friend_to_remove.lower() == 'back':
                return

            if friend_to_remove in self.graph[user_id]:
                self.graph[user_id].remove(friend_to_remove)
                self.graph[friend_to_remove].remove(user_id)
                print(f"Friend {friend_to_remove} removed successfully!")
                self.save_data()
                return
            else:
                print(f"You are not friends with {friend_to_remove}!")
                return

    def see_friends(self, user_id):#O(n) iterates over all friends which has n friends
        while True:
            print("\nYour friends:")
            for friend in self.graph[user_id]:
                print(friend)
            print("\nEnter 'back' to go back:")
            choice = input("Enter your choice: ")
            if choice.lower() == 'back':
                return

    def see_all_users(self):#O(n) iterates over all users which has n users
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

    def suggest_friends(self, user_id):#O(n^2) iterates over all user profiles to build a graph, and then finds all simple paths of length 2
        suggested_friends = []
        user_profile = self.user_profiles[user_id]
        user_hobbies = set(user_profile['hobbies'])
        user_interests = set(user_profile['interests'])

        G = nx.Graph()
        G.add_nodes_from(self.user_profiles)

        for user1, profile1 in self.user_profiles.items():
            for user2, profile2 in self.user_profiles.items():
                if user1 != user2 and user1 not in self.graph[
                        user2] and user2 not in self.graph[user1]:
                    hobbies_intersection = user_hobbies & set(
                        profile2['hobbies'])
                    interests_intersection = user_interests & set(
                        profile2['interests'])
                    G.add_edge(user1,
                               user2,
                               weight=len(hobbies_intersection) +
                               len(interests_intersection))

        for node in G.nodes:
            if node != user_id:
                for path in nx.all_simple_paths(G, user_id, node):
                    if len(path) == 2:
                        hobbies_reason = ', '.join(hobbies_intersection)
                        interests_reason = ', '.join(interests_intersection)
                        reason = f"Shared hobbies: {hobbies_reason}, Shared interests: {interests_reason}"
                        suggested_friends.append((node, reason))
                        break

        return suggested_friends

    def recommend_friends(self, user_id):#O(n^2) calls suggest_friends, which has O(n^2) as big O notation
        while True:
            print("\nGet friend recommendations:")
            suggested_friends = self.suggest_friends(user_id)

            if suggested_friends:
                print("Friend recommendations:")
                for i, (friend, reason) in enumerate(suggested_friends):
                    print(f"{i+1}. {friend} - {reason}")
            else:
                print("No friend recommendations!")

            print("\nEnter 'back' to go back:")
            choice = input("Enter your choice: ")
            if choice.lower() == 'back':
                return

    def send_see_message(self, user_id):#O(n) iterates over all the messages where having n messages
        while True:
            print("\nSend/Receive messages:")
            to_user = input(
                "Enter the username of the user you want to send a message to (or 'back' to go back): "
            )
            self.save_data()

            if to_user.lower() == 'back':
                return

            for uid in self.graph:
                if uid.lower() == to_user.lower():
                    to_user = uid
                    break
            else:
                print(f"User {to_user} does not exist or you are not friends!")
                continue

            if to_user in self.graph[user_id]:
                while True:
                    print("\nYour conversation with", to_user)
                    if user_id in self.messages and to_user in self.messages[
                            user_id]:
                        for message in self.messages[user_id][to_user]:
                            print(message)
                    print("\nEnter '.' to go back:")
                    message = input("Enter your message: ")
                    if message.lower() == '.':
                        break
                    if user_id not in self.messages:
                        self.messages[user_id] = {}
                    if to_user not in self.messages[user_id]:
                        self.messages[user_id][to_user] = []
                    self.messages[user_id][to_user].append(f"You: {message}")
                    if to_user not in self.messages:
                        self.messages[to_user] = {}
                    if user_id not in self.messages[to_user]:
                        self.messages[to_user][user_id] = []
                    self.messages[to_user][user_id].append(
                        f"From {user_id}: {message}")
                    print(f"Message sent to {to_user} successfully!")
                    self.save_data()
            else:
                print(f"You are not friends with {to_user}")
                return

    def post_tweet(self, user_id):#O(1)
        while True:
            print("\nPost a tweet:")
            tweet = input("Enter your tweet (or 'back' to go back): ")

            if tweet.lower() == 'back':
                return

            if user_id not in self.tweets:
                self.tweets[user_id] = []
            self.tweets[user_id].append(tweet)
            if user_id not in self.likes:
                self.likes[user_id] = {}
                self.likes[user_id][tweet] = {'whistles': 0, 'eows': 0}

            print(f"Tweet posted successfully!")
            self.save_data()
            return

    def view_tweets(self, user_id):#O(n) iterates over all tweets which contain n tweets
        while True:
            print("\nView tweets:")
            for user, tweets in self.tweets.items():
                for tweet in tweets:
                    print(f"{user}: {tweet}")
                    if user in self.likes and tweet in self.likes[user]:

                        print(
                            f"Whistles: {self.likes[user][tweet]['whistles']}, Meows: {self.likes[user][tweet]['eows']}"
                        )
                    else:
                        print("No likes or dislikes yet!")
            print(
                "\nEnter 'back' to go back, or 'like' or 'dislike' to interact with a tweet: "
            )
            choice = input("Enter your choice: ")
            if choice.lower() == 'back':
                return
            elif choice.lower() == 'like':

                tweet_user = input(
                    "Enter the username of the tweet you want to like: ")

                tweet_text = input(
                    "Enter the text of the tweet you want to like: ")

                if tweet_user in self.tweets and tweet_text in self.tweets[
                        tweet_user]:

                    if tweet_user in self.likes and tweet_text in self.likes[
                            tweet_user]:

                        self.likes[tweet_user][tweet_text]['whistles'] += 1

                    else:

                        self.likes[tweet_user][tweet_text] = {
                            'whistles': 1,
                            'eows': 0
                        }

                    print(f"You whistled at {tweet_user}'s tweet!")
                    self.save_data()

                else:

                    print(f"Tweet not found!")

            elif choice.lower() == 'dislike':

                tweet_user = input(
                    "Enter the username of the tweet you want to dislike: ")

                tweet_text = input(
                    "Enter the texbact of the tweet you want to dislike: ")

                if tweet_user in self.tweets and tweet_text in self.tweets[
                        tweet_user]:

                    if tweet_user in self.likes and tweet_text in self.likes[
                            tweet_user]:

                        self.likes[tweet_user][tweet_text]['eows'] += 1

                    else:

                        self.likes[tweet_user][tweet_text] = {
                            'whistles': 0,
                            'eows': 1
                        }

                    print(f"You meowed at {tweet_user}'s tweet!")
                    self.save_data()

                else:

                    print(f"Tweet not found!")

            else:

                print("Invalid choice!")

    def home(self, user_id):#O(1)
        while True:
            print("\nHome")
            print("1. Send a friend request")
            print("2. Accept a friend request")
            print("3. See friends")
            print("4. See all users")
            print("5. Get friend recommendations")
            print("6. Remove a friend")
            print("7. Send/see a message")
            print("8. Post a tweet")
            print("9. View tweets")
            print("10. Logout")
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
                self.send_see_message(user_id)
            elif choice == "8":
                self.post_tweet(user_id)
            elif choice == "9":
                self.view_tweets(user_id)
            elif choice == "10":
                break
            else:
                print("Invalid choice!")

    def run(self):#O(1)
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
