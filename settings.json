#-----------------------------------------------------------------------
# author: Alex le
# purpose: to book appointments and to view up and coming appointment
# as well to suggested times to avoid and book
# Date: 12/18/2025
# program_name:SchedulerX
# ----------------------------------------------------------------------

from flask import Flask, render_template, request, redirect, url_for
from collections import Counter
import json
import datetime
import os
from sklearn.tree import DecisionTreeClassifier,export_text
from collections import Counter

app = Flask(__name__)
work_days = [8,9,10,11,13,14,15] # working hours
words_days_str = ['Monday','Tuesday','Wednesday','Thursday','Friday']

# json logic of creating a file if not there, loading and saving
APPOINTMENTS_FILE = 'appointments.json'
# create a json file if it doesn't exist
if not os.path.exists(APPOINTMENTS_FILE):
    with open(APPOINTMENTS_FILE, 'w') as f:
        json.dump([], f)

# load appointments from json file
def load_appointments():
    with open(APPOINTMENTS_FILE, 'r') as f:
        return json.load(f)
#save a dict to the json file
def save_appointments(appointments):
    with open(APPOINTMENTS_FILE, 'w') as f:
        json.dump(appointments, f, indent=3)

# render booking page
@app.route('/booking')
def booking():
    appointments = load_appointments()
    list_suggested = get_suggested_times(appointments)
    return render_template('appointment.html', 
                          suggested=list_suggested )

# event listener and new appointment creator
@app.route('/submit', methods=['POST'])
def submit():
    #fetch form
    data = request.form
    
    # create the new dict
    new_appointment = {
        'name': data['name'],
        'date': data['date'],
        'time': data['time'],
    }# load existing appointments
    

    appointments = load_appointments()
    data_hour = datetime.datetime.strptime(data["time"], "%H:%M").hour
    data_minute = datetime.datetime.strptime(data["time"], "%H:%M").minute
    data_day = datetime.datetime.strptime(data["date"], "%Y-%m-%d").weekday()
    valid_hours = {8, 9, 10, 11, 13, 14, 15}

    list_suggested = get_suggested_times(appointments)

    # check if appointment is valid
    if data['time'] == "" or data["date"] == "" or data['name'] =="":
        return render_template('appointment.html',feedback = 'none',
                               suggested=list_suggested)
    
    # check if it's within the time table

    #check the days 0-(monday) -4(friday)
    if data_day > 4:
        return render_template(
            'appointment.html',
            feedback='out_of_time',
            suggested=list_suggested
        )

    #check the hours using military time and a time's list
    if data_hour not in valid_hours or data_minute != 0:
        return render_template(
            'appointment.html',
            feedback='out_of_time',
            suggested=list_suggested
        )


    # check if appointment is filled
    for val in appointments:
        if data['time'] == val['time'] and data['date'] == val['date']:
            return render_template('appointment.html',feedback = 'booked',
                                   suggested=list_suggested)


    appointments.append(new_appointment)
    save_appointments(appointments)
    return render_template('index.html',app = appointments, status = True)

# render homepage
@app.route('/')
def home():
    return render_template('index.html', app=load_appointments(),status = False)


# get's suggested time through analyzed times
def get_suggested_times(appointments):
    
    slots = []
    for day in range(0, 5):
        for hour in work_days:
                slots.append((day, hour))

    # init count at zero for every time slot
    frequency_map = {slot: 0 for slot in slots}

    # count frequency of appointments
    for i in appointments:
        temp_hour = datetime.datetime.strptime(i["time"], "%H:%M").hour
        temp_day = datetime.datetime.strptime(i["date"], "%Y-%m-%d").weekday()
        
        if (temp_day, temp_hour) in frequency_map:
            frequency_map[(temp_day, temp_hour)] += 1

    lowest_val = min(frequency_map.values())
    print("Lowest value:", lowest_val)

    # convert dict to list
    feature_x = list(frequency_map.keys())
    feature_y = list(frequency_map.values())    
        
    
    model = DecisionTreeClassifier()
    model.fit(feature_x,feature_y)
    
    print("Training features:", feature_x)
    print("Training labels:", feature_y)
   
    # predict and suggest monday - friday between 8am -11am, 1pm -3pm for 
    top_5 = []
    for n in range(0,5):
        for i in work_days:  
            
            predict = model.predict([[n, i]])   
            print(export_text(model, feature_names=["day", "hour"]))
            day = words_days_str[n]
            time = f"{i:02d}:00"
            top_5.append({'date': day, 'time': time,'score': predict})
            
    # sort by score, date, time in that order
    top_5.sort(key=lambda x: (x['score'], x['date'], x['time']))
    print("Top 5 suggested times:", top_5)
    return top_5[:5]       
    



if __name__ == '__main__':
    app.run(debug=True)
