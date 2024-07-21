# Back-End-boxzi

from flask import Flask, request, jsonify  
import sqlite3  
from datetime import datetime  

app = Flask(__name__)  

# Create a connection to SQLite database  

conn = sqlite3.connect('followers.db')  
cursor = conn.cursor()  

# Create a table to store user data  

cursor.execute('''CREATE TABLE IF NOT EXISTS users  
                (id INTEGER PRIMARY KEY, username TEXT, followers TEXT, following TEXT)''')  
conn.commit()  

# API for following a user  

@app.route('/follow', methods=['POST'])  
def follow_user():  
    data = request.get_json()  
    follower = data.get('follower')  
    following = data.get('following')  

    cursor.execute("SELECT followers FROM users WHERE username = ?", (following,))  
    current_followers = cursor.fetchone()[0]  
    
    if current_followers:  
        current_followers += f",{follower}"  
    else:  
        current_followers = follower  

    cursor.execute("UPDATE users SET followers = ? WHERE username = ?", (current_followers, following))  
    conn.commit()  
    
    return jsonify({"message": f"{follower} is now following {following}"}), 200  

# API for unfollowing a user  

app.route('/unfollow', methods=['POST'])  
def unfollow_user():  
    data = request.get_json()  
    follower = data.get('follower')  
    following = data.get('following')  

    cursor.execute("SELECT followers FROM users WHERE username = ?", (following,))  
    current_followers = cursor.fetchone()[0].split(',')  
    
    current_followers.remove(follower)  
    updated_followers = ','.join(current_followers)  

    cursor.execute("UPDATE users SET followers = ? WHERE username = ?", (updated_followers, following))  
    conn.commit()  
    
    return jsonify({"message": f"{follower} has unfollowed {following}"}), 200  

# API to get daily followers count of a user  

@app.route('/followers_count/<username>', methods=['GET'])  
def get_followers_count(username):  
    cursor.execute("SELECT followers FROM users WHERE username = ?", (username,))  
    followers = cursor.fetchone()[0]  
    
    if not followers:  
        return jsonify({"message": "User has no followers."}), 200  

    daily_followers_count = len(followers.split(','))  
    
    return jsonify({"followers_count": daily_followers_count}), 200  

# API to get common followers of two users  

@app.route('/common_followers', methods=['POST'])  
def get_common_followers():  
    data = request.get_json()  
    user1 = data.get('user1')  
    user2 = data.get('user2')  

    cursor.execute("SELECT followers FROM users WHERE username = ?", (user1,))  
    followers1 = set(cursor.fetchone()[0].split(','))  

    cursor.execute("SELECT followers FROM users WHERE username = ?", (user2,))  
    followers2 = set(cursor.fetchone()[0].split(','))  

    common_followers = followers1.intersection(followers2)  
    
    return jsonify({"common_followers": list(common_followers)}), 200  

if __name__ == '__main__':  
    app.run(debug=True) 
