# Project
import streamlit as st
import pandas as pd
import numpy as np
import calendar
from datetime import datetime
import time

def run_hr(session_hr,session_mins):
    countdown_mins = session_hr * 60*60
    countdown_seconds = session_mins * 60
    total_sec=countdown_seconds+countdown_mins
    with st.empty():
        for hr in range(total_sec//60,0,-1):
            hr-=1    
            for i in range(60,0,-1):
                st.write(f"⏳ {hr} minutes {i} seconds")
                time.sleep(1)
    with st.empty():
        for mins in range(countdown_seconds):
            st.write(f"⏳ {mins} seconds have passed")
            time.sleep(1)

    st.success('Time is up! Take a break.')
    st.balloons()
# Function to cancel the Pomodoro session
def cancel_pomodoro():
    st.error('Pomodoro session canceled.')

# Creating the Streamlit app

def delete_all_tasks():
    """Delete all tasks from the tasks DataFrame and the tasks.csv file."""
    tasks = pd.DataFrame({"Task": [], "Date": [], "Priority": [], "Status": [], "Check": []})
    tasks.to_csv("tasks.csv", index=False)
    st.experimental_rerun()

tab1, tab2 = st.tabs(["TO DO APP","POMODORO TIMER"])
with tab1:
    st.title("To-Do App")

# Load existing tasks
    try:
        tasks = pd.read_csv("tasks.csv")
    except:
        tasks = pd.DataFrame({"Task": [], "Date": [], "Priority": [], "Status": [], "Check": []})

    # Remove duplicate tasks
    tasks = tasks.drop_duplicates()

    col1, col2= st.columns(2)
    # Create new task or update existing task
    with col1:
        with st.form(key='task_form'):
            new_task = st.text_input("Enter new task")
            task_date = st.date_input("Select task date")
            task_priority = st.selectbox("Select task priority", ["Low", "Medium", "High"])
            task_status = st.selectbox("Select task status", ["Not Started", "In Progress", "Completed"])
            submitted = st.form_submit_button('Add Task')

        if submitted:
            existing_task = tasks.loc[(tasks["Task"] == new_task) & (tasks["Date"] == task_date) & (tasks["Priority"] == task_priority) & (tasks["Status"] == task_status)]
            if existing_task.empty:
                new_row = pd.DataFrame({"Task": [new_task], "Date": [task_date], "Priority": [task_priority], "Status": [task_status], "Check": ["False"]})
                tasks = pd.concat([tasks, new_row], ignore_index=True)
                tasks.to_csv("tasks.csv", index=False)
                st.success("Task added successfully")

    # Add a Clear button at the top of the application
    if st.button("Clear"):
        delete_all_tasks()
    # Display existing tasks
    if tasks.shape[0] > 0:
        for index, row in tasks.iterrows():
            st.checkbox(f"{row['Task']} ({row['Date']})", key=f"checkbox{index}")
    else:
        st.write("No tasks to display")

    with col2:
        # Update tasks in calendar
        year = datetime.now().year
        month = datetime.now().month
        calendar_html = calendar.HTMLCalendar(firstweekday=calendar.SUNDAY).formatmonth(year, month, tasks.to_dict('records'))
        st.write(calendar_html, unsafe_allow_html=True)
        st.text("")
        # Set up the iframe
        iframe_html = """
                        <iframe style="border-radius:12px" src="https://open.spotify.com/embed/playlist/1samlRVE4AvI9UJJYQ7XEU?utm_source=generator" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>"""

        # Display the iframe in the Streamlit app
        st.markdown(iframe_html, unsafe_allow_html=True)

        iframe_html = """
                        <iframe style="border-radius:12px" src="https://indify.co/widgets/live/progressBar/MFt0fceI2FQ5GQvVkPOv" width="100%" height="152" frameBorder="0" allowfullscreen="" allow="autoplay; clipboard-write; encrypted-media; fullscreen; picture-in-picture" loading="lazy"></iframe>"""

        # Display the iframe in the Streamlit app
        st.markdown(iframe_html, unsafe_allow_html=True)


with tab2:
    st.title("Pomodoro Timer")
    hr = st.text_input('Enter the duration (in hours) for your Pomodoro session:')
    mins = st.text_input('Enter the duration (in minutes) for your Pomodoro session:')

    if st.button('Start'):
        sess_mins = int(mins)
        sess_hr=int(hr)
        print(sess_mins,sess_hr)
        run_hr(sess_hr,sess_mins)

    if st.button('Cancel'):
        cancel_pomodoro()
