
import sqlite3
import random

# TABLE1
conn = sqlite3.connect('miniassignment.db')
cursor = conn.cursor()
cursor.execute('''
        CREATE TABLE IF NOT EXISTS user(
               user_id   INTEGER PRIMARY KEY AUTOINCREMENT,  
               username   VARCHAR(20) UNIQUE,
               password   VARCHAR(20),
               role    TEXT
               )
    ''')
# TABLE2
cursor.execute('''
        CREATE TABLE IF NOT EXISTS member(
               member_id   INTEGER PRIMARY KEY AUTOINCREMENT,
               user_id    INTEGER,
               member_name   TEXT UNIQUE,
               phone_no  INTEGER,
               age   INTEGER,
               height   REAL,
               weight   REAL,
               goal   TEXT,
               calories INTEGER, 
               trainer_id  INTEGER,
               FOREIGN KEY (user_id) REFERENCES user(user_id)
               )
    ''')
# TABLE3
cursor.execute('''
        CREATE TABLE IF NOT EXISTS subscriptions(
               sub_id      INTEGER PRIMARY KEY AUTOINCREMENT,
               member_id   INTEGER,
               plan   VARCHAR(10),
               fee          INTEGER,
               FOREIGN KEY (member_id) REFERENCES member(member_id)
               
               )
    ''')

conn.commit()

cursor.execute('''
        INSERT OR IGNORE INTO user(username, password, role)
               VALUES('noor', 'noor12345', 'Manager')
    ''')
conn.commit()


def register():
    conn = sqlite3.connect('miniassignment.db')
    cursor = conn.cursor()
    username = input('enter new username:- ').strip()
    if len(username)>20:
        print('username must contain only 20 characters')
        return
    password = input('enter new  password:- ')
    if len(password)<8:
        print('password should have atleast 8 characters')
        return
    role = input('recognise who you are: Trainer / Member\nChoose above options:- ').capitalize().strip()
    if role not in ['Trainer', 'Member']:
        print('only Trainer or Members can register') 
        return
    
    cursor.execute('''
        INSERT INTO user(username, password, role)
                VALUES(?,?,?)
    ''',(username, password, role))
    conn.commit()

    if role == 'Trainer':
        print('<=============================>')
        print('Your registration as a Trainer successful')
        print('<================================>')
    
    elif role == 'Member':
        cursor.execute('''
        SELECT user_id FROM user WHERE username = ?
        ''',(username, ))
        user_id = cursor.fetchone()[0]
        name = input('enter the name:- ')
        phone = int(input('enter the phone no.:- '))
        cursor.execute('''
                INSERT INTO member(user_id, member_name, phone_no)
                        VALUES(?,?,?)
                    ''',(user_id, name, phone))
        conn.commit()
        print('<------------------------>')
        print('Registration Successfull')
        print('<=========================>')
    else:
        print('<---------------------->')
        print('Registration is not successfull, please try again')



def login():
    conn = sqlite3.connect('miniassignment.db')
    cursor = conn.cursor()
    username = input('enter new username:- ').strip()
    if len(username)>20:
        print('username must contain only 20 characters')
        return
    password = input('enter new  password:- ').strip()
    if len(password)<8:
        print('password should have atleast 8 characters')
        return
    elif not password:
        print('password cannot be empty')
    role = input('recognise who you are: Manager  /  Trainer  /  Member\n \n<----------------------------->\nCHOOSE THE ABOVE OPTION:- ').capitalize().strip()

    cursor.execute('''
        SELECT user_id FROM user WHERE username = ? AND password = ? AND role=?
    ''',(username, password, role))
    user = cursor.fetchone()
    if not user:
        print('login failed...user not found!!!')
    if user:
        user_id = user[0]
        print('your login has been successfully completed')
        if role == 'Manager':
            manager_panel()
        elif role == 'Trainer':
            Trainer_panel(user_id)
        elif role == 'Member':
            member_panel(user_id)   

def manager_panel():
    conn = sqlite3.connect('miniassignment.db')
    cursor = conn.cursor()
    while True:
        print('<----------------------------->')
        print('EXPLORE MANAGER PANEL')
        print('<==============================>')
        choice = int(input('\n\n1. View Members\n2. View Trainers \n3. View Subscriptions \n4. View Total Revenue \n5. Delete Member \n6. Add Members \n7. Add Trainers \n8. Logout \nCHOOSE YOUR OPTION:- '))
        if choice == 1:
            cursor.execute('''
            SELECT * FROM member
        ''')
            z = cursor.fetchall()
            for i in z:
                print(f'member id: {i[0]} membername: {i[2]} phone no. {i[3]}')
        elif choice == 2:
             cursor.execute('''
                SELECT * FROM user WHERE role = 'Trainer'
            ''')
             z = cursor.fetchall()
             for i in z:
                print(f'Trainer id: {i[0]}   Trainer name: {i[1]}')
        elif choice == 3:
            cursor.execute('''
                    SELECT member_id, plan, fee FROM subscriptions
                ''')
            data = cursor.fetchall()
            for i in data:
                memberid = i[0]
                subplan = i[1]
                subfee = i[2]
                cursor.execute('''
                    SELECT member_name FROM member WHERE member_id = ?
                    ''',(memberid, ))
                name = cursor.fetchone()
                print('NAME | PLAN | FEE')
                print(name[0], subplan, subfee)
        elif choice == 4:
            cursor.execute('''
                SELECT SUM(fee) FROM subscriptions
                ''')        
            total = cursor.fetchone()[0]
            if total is None:
                total = 0
            print('Total fee received:- ', total)

        elif choice == 5:
            member_id = input('enter the member id which you want to remove:- ')
            k = input('Are you sure you want to remove the user from the gym app?\ny/n:- ')
            if k == 'y':
                cursor.execute('''
                    SELECT * FROM member WHERE member_id = ?
                ''',(member_id, ))
                if not cursor.fetchone():
                    print('\n<----------------->\nmember id not found')

                else:
                    cursor.execute('''
                    DELETE FROM member WHERE member_id = ?
                    ''',(member_id, ))
                    print(f'{member_id} member is successfully removed')
                    conn.commit()
            else:
                print(f'removal of {member_id} member is cancelled')
                print('<---------------------------->')

        elif choice == 6:
            username = input('Enter the member name:- ')
            password = input('Enter the member password:- ')
            try:
                cursor.execute('''
                INSERT INTO user(username, password, role)
                               VALUES(?,?,'Member')
                ''',(username, password))
                conn.commit()

                cursor.execute('''
                SELECT user_id FROM user WHERE username = ?
                ''',(username, ))
                user_id = cursor.fetchone()

                member_name = input('Enter name of the user:- ')
                phone_no = int(input('Enter phone no:- '))
                cursor.execute('''
                INSERT INTO member(user_id, member_name, phone_no)
                               VALUES(?,?,?)
                ''',(user_id[0], member_name, phone_no))
                conn.commit()
                print('\n----------------------------\nMEMBER ADDED SUCCESSFULLY')
            except:
                print('username exists')

        elif choice == 7:
            username = input('Enter trainer name:- ')
            trainer_password = input('Enter trainer password:- ')
            try:
                cursor.execute('''
                INSERT INTO user(username, password, role)
                           VALUES(?,?,'Trainer')  
                ''',(username, trainer_password))
                conn.commit()
                print('\n--------------------------\nTrainer Added')
            except:
                print('username already exists')

        elif choice == 8:
            print('You are successfully logged out')
            print('<---------------------------------->')
            break

def get_plan():
    data = int(input('Choose the plan options below:\n1. one month plan \n2. Three month plan \n3. Six month plan\n4. One year plan\nChoose one of the subsciption plan:-'))
    if data == 1:
        return'One month plan',1000
    elif data == 2:
        return'Three month plan',2500
    elif data == 3:
        return'Six month plan',4500
    elif data == 4:
        return'One year plan',8000
    else:
        print('invalid choice')
        return None, None
    
def Trainer_panel(Trainer_id):
        conn = sqlite3.connect('miniassignment.db')
        cursor = conn.cursor()
        while True:
            selection = int(input('\n----Trainer Panel----\n-----------------\n1. View Members \n2. View Subscriptions \n3. update member plan \n4. Delete member plan\n5. Logout\nCHOOSE ONE OF THE OPTION:- '))
            if selection == 1:
             try:  
                    cursor.execute('''
                    SELECT member_id, member_name, phone_no FROM member WHERE Trainer_id = ?
                    ''',(Trainer_id, ))
                    data = cursor.fetchall()
                    for i in data:
                        print(f'member_id: {i[0]}   member_name:{i[1]} ')
             except:
                print('member details not found')
            elif selection == 2:
                try:
                        cursor.execute('''
                    SELECT member_id, plan, fee FROM subscriptions
                    ''')
                        data = cursor.fetchall()
                        for i in data:
                            member_id = i[0]
                            plan = i[1]
                            fee = i[2]
                        cursor.execute('''
                    SELECT member_name FROM member where member_id = ? AND trainer_id =?    
                    ''',(member_id, Trainer_id))
                        name = cursor.fetchone()
                        if name:
                            print(name[0], plan, fee)
        
                except: 
                    print('subcription details not found')

            elif selection == 3:
                member_id = int(input('Enter the member id:- '))
                plan, fee = get_plan()
                cursor.execute('''
                UPDATE subscriptions SET plan = ?, fee=? WHERE member_id = ?
                ''',(plan, fee, member_id))
                conn.commit()
                print('New plan UPDATED')
     
            elif selection == 4:
                member_id = int(input('Enter the member id of the member that subscription need to be removed:- '))
                cursor.execute('''
                DELETE FROM subscriptions WHERE member_id = ?
            ''',(member_id, ))
                conn.commit()
                print('subscription removed successfully')
            elif selection ==5:
                print('You are successfully Logged Out')
                break
            else:
                print('No such responses accepted')

def calories_calculator(weight, goal):
    maintenance = weight * 30
    if goal == 'Gain':
        return maintenance + 500
    elif goal == 'Loss':
        return maintenance - 500
    else:
        return maintenance 


def member_panel(user_id):
        conn = sqlite3.connect('miniassignment.db')
        cursor = conn.cursor()
        cursor.execute('''
            SELECT member_id, age FROM member WHERE user_id = ?
        ''',(user_id, ))
        result = cursor.fetchone()
        if not result:
            print('Member not found')
            return
        member_id, age = result
        if age is None:
            age = int(input('Enter your age:- '))
            height = float(input('Enter your Height:- '))
            weight = float(input('Enter your weight:- '))
            goal = input('Enter your goal (Gain/Loss):- ').capitalize()
            calories = calories_calculator(weight, goal)
            cursor.execute('''
                UPDATE member SET age = ?, height = ?, weight = ?, goal = ?, calories=? WHERE member_id = ?
            ''',(age, height, weight, goal, calories, member_id))
            conn.commit()
            print('calories:- ', calories)

        while True:
            select = int(input('<---------WELCOME TO MEMBER PANEL------------>\n===========================\n1. View Body Details\n2. Purchase Plan \n3. Update My plan\n4. View Trainer\n5. Logout\nSELECT ANY ONE OPTION:- '))
            if select == 1:
                cursor.execute('''
                    SELECT age, height, weight, goal, calories FROM member WHERE member_id = ?
                ''',(member_id, ))
                f = cursor.fetchone()
                print(f'age: {f[0]} \nHeight: {f[1]} \nWeight: {f[2]} \nGoal: {f[3]} \nCalories: {f[4]}')
                print('==================================================================================')
            
            elif select == 2:
                plan, fee = get_plan()
                if plan:
                    cursor.execute('''
                    INSERT INTO subscriptions(member_id, plan, fee)
                                   VALUES(?,?,?)
                    ''',(member_id, plan, fee))
                    
                    cursor.execute('''
                    SELECT user_id FROM user WHERE role = 'Trainer'
                    ''')
                    trainer = cursor.fetchall()
                    trainerid = random.choice(trainer)[0]
                    cursor.execute('''
                    UPDATE member SET trainer_id = ? WHERE member_id = ?
                    ''',(trainerid, member_id))
                    conn.commit()
                    print('<============================>\nCONGRATS, PLAN HAS BEEN SUCCESSFULLY PURCHASED\n TRAINER ALLOCATED SUCCESSFULLY')
                    print('================================================================================================================')   

            elif select == 3:
                plan, fee = get_plan()
                cursor.execute('''
                UPDATE subscriptions SET plan = ?, fee = ? WHERE member_id = ? 
                ''',(plan, fee, member_id))
                conn.commit()
                print('----------------------------------------')
                print('YOUR PLAN HAS BEEN SUCCESSFULLY UPDATED')
                print('=========================================')

            elif select == 4:
                cursor.execute('''
                SELECT trainer_id FROM member WHERE member_id = ? 
                ''',(member_id, ))
                trainer_id = cursor.fetchone()

                if trainer_id:
                    cursor.execute('''
                    SELECT username FROM user WHERE user_id = ?
                    ''',(trainer_id[0], ))
                    trainer = cursor.fetchone()
                    print(f'Trainer:- {trainer[0]}')
                else:
                    print('-----------------------------------')
                    print('No trainer available at the moment')
                    print('===================================')
            
            elif select == 5:
                print('------------------')
                print('LOGOUT SUCCESSFULL')
                print('===================')
                break
            else:
                print('--------------------------')
                print('No such response accepted')
                print('==========================')
                

                            
    
        
def main():
    while True:
        print('<----Welcome to GYM MANAGEMENT APP---->')
        z = int(input('choose the below options:\n1. REGISTER \n2. LOGIN \n3.exit\n<---------->\nChoose one of the above options:- '))
        if z == 1:
            register()
        elif z == 2:
            login()
        elif z == 3:
            print('good bye')
            break
        else:
            print('this option not receivable')        
main()

