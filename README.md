# KLL-portal
import tkinter as tk
from tkinter import ttk, messagebox
import datetime

class AppController:
    def __init__(self, root, container):
        self.root = root
        self.container = container

        self.users = {
            "students": {
                "student1": "password123",
                "student2": "securepass"
            },
            "teachers": {
                "teacher1": "teachpass"
            }
        }

        self.frames = {}
        self.grades = {}

        login_frame = LoginPage(self.container, self)
        self.frames[LoginPage] = login_frame
        login_frame.pack(fill="both", expand=True)
        
        self.show_frame(LoginPage)
        
        
    def show_frame(self, page_class):
        frame = self.frames[page_class]
        frame.tkraise()

    def add_frame(self, page_class, frame_instance):
        self.frames[page_class] = frame_instance
        frame_instance.pack(fill="both", expand=True)
        self.show_frame(page_class)

class LoginPage(tk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent, bg="#f7f1e3")
        self.controller = controller
        self.users = controller.users

        # Pinakang pamagat ng portal
        tk.Label(
            self,
            text="🏫 Welcome to KLL School Portal",
            font=("Segoe UI", 24, "bold"),
            bg="#4e342e",
            fg="white",
            pady=20
        ).pack(fill="x")
        

        # dito maglologin
        self.frame = tk.Frame(self, bg="#ffffff", bd=2, relief="groove")
        self.frame.place(relx=0.5, rely=0.5, anchor="center", width=500, height=320)

        tk.Label(self.frame, text="LOGIN", font=("Segoe UI", 18, "bold"), bg="#ffffff", fg="#4e342e").pack(pady=(20, 10))

        self.add_input("Username:", is_password=False)
        self.add_input("Password:", is_password=True)

        btn_frame = tk.Frame(self.frame, bg="#ffffff")
        btn_frame.pack(pady=20)

        ttk.Button(btn_frame, text="Login", command=self.login, width=18).grid(row=0, column=0, padx=8)
        ttk.Button(btn_frame, text="Forgot Password", command=self.forgot_password, width=18).grid(row=0, column=1, padx=8)
        ttk.Button(btn_frame, text="Register", command=self.open_category_selection, width=40).grid(row=1, column=0, columnspan=2, pady=10)

    def add_input(self, label_text, is_password=False):
        frame = tk.Frame(self.frame, bg="#ffffff")
        frame.pack(pady=8)
        tk.Label(frame, text=label_text, font=("Segoe UI", 12), bg="#ffffff").pack(anchor="w", padx=10)
        entry = ttk.Entry(frame, font=("Segoe UI", 12), width=35, show="*" if is_password else "")
        entry.pack(padx=10)
        if "Username" in label_text:
            self.username_entry = entry
        elif "Password" in label_text:
            self.password_entry = entry

    def login(self):
        username = self.username_entry.get().strip()
        password = self.password_entry.get().strip()

        student_data = self.users["students"].get(username)
        if isinstance(student_data, dict) and student_data.get("password") == password:
            login_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            self.controller.login_time = login_time
            self.controller.current_user = username
            messagebox.showinfo("Login Successful", f"Welcome, {username}! You are logged in as a Student.\nLogin Time: {login_time}")
            self.pack_forget()
            if StudentPortal not in self.controller.frames:
                frame = StudentPortal(self.controller.container, self.controller)
                self.controller.add_frame(StudentPortal, frame)
            else:
                frame = self.controller.frames[StudentPortal]
            frame.show_dashboard()
            return

        teacher_data = self.users["teachers"].get(username)
        if isinstance(teacher_data, dict) and teacher_data.get("password") == password:
            login_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
            self.controller.login_time = login_time
            self.controller.current_user = username
            messagebox.showinfo("Login Successful", f"Welcome, {username}! You are logged in as a Teacher.\nLogin Time: {login_time}")
            self.pack_forget()
            if TeacherPortal not in self.controller.frames:
                frame = TeacherPortal(self.controller.container, self.controller)
                self.controller.add_frame(TeacherPortal, frame)
            else:
                frame = self.controller.frames[TeacherPortal]
            frame.show_dashboard()
            return

     
        messagebox.showerror("Login Failed", "Invalid username or password.")
    
    def forgot_password(self):
        messagebox.showinfo("Forgot Password", "Please contact admin@portal.com for assistance.")

    def open_category_selection(self):
        # Hide the login frame
        self.frame.place_forget()

        # Overlay frame for category selection
        self.category_overlay = tk.Frame(self, bg="#f7f1e3", bd=2, relief="ridge")
        self.category_overlay.place(relx=0.5, rely=0.5, anchor="center", width=400, height=280)

        tk.Label(
            self.category_overlay,
            text="👤 Select Registration Type",
            font=("Segoe UI", 16, "bold"),
            bg="#4e342e",
            fg="white",
            pady=15
        ).pack(fill="x")

        content_frame = tk.Frame(self.category_overlay, bg="#f7f1e3")
        content_frame.pack(pady=30)

        ttk.Button(content_frame, text="Register as Student",
                   command=lambda: self.open_register_frame("Student"), width=30).pack(pady=10)
        ttk.Button(content_frame, text="Register as Teacher",
                   command=lambda: self.open_register_frame("Teacher"), width=30).pack(pady=10)
        ttk.Button(content_frame, text="Cancel",
                   command=self.close_category_overlay, width=30).pack(pady=10)

    def close_category_overlay(self):
        if hasattr(self, 'category_overlay'):
            self.category_overlay.destroy()
        # Show the login frame again
        self.frame.place(relx=0.5, rely=0.5, anchor="center", width=500, height=320)

    def open_register_frame(self, user_type):
        self.close_category_overlay()
        self.pack_forget()
        if user_type == "Student":
            if StudentRegistrationForm not in self.controller.frames:
                frame = StudentRegistrationForm(self.controller.container, self.controller)
                self.controller.add_frame(StudentRegistrationForm, frame)
            else:
                self.controller.show_frame(StudentRegistrationForm)
        else:
            if TeacherRegistrationForm not in self.controller.frames:
                frame = TeacherRegistrationForm(self.controller.container, self.controller)
                self.controller.add_frame(TeacherRegistrationForm, frame)
            else:
                self.controller.show_frame(TeacherRegistrationForm)

class TeacherPortal(tk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent, bg="#e0f7fa")
        self.controller = controller

        # Header
        tk.Label(
            self,
            text="🎓 KLL Teacher Management System",
            font=("Segoe UI", 24, "bold"),
            bg="#00796b",
            fg="white",
            pady=20
        ).pack(fill="x")

        # Top navigation buttons
        button_frame = tk.Frame(self, bg="#e0f7fa")
        button_frame.pack(fill="x", pady=10)

        buttons = [
            ("📊 DASHBOARD", self.show_dashboard),
            ("📁 MENU", self.show_menu),
            ("📰 NEWS", self.show_news),
            ("📅 EVENTS", self.show_events),
            ("📚 LESSON PLAN", self.show_lesson_plan),
            ("❌ LOGOUT", self.back_to_login)
        ]

        for text, command in buttons:
            bg = "#00796b" if "LOGOUT" not in text else "#d32f2f"
            tk.Button(
                button_frame,
                text=text,
                command=command,
                bg=bg,
                fg="white",
                font=("Segoe UI", 10, "bold"),
                activebackground="#004d40",
                activeforeground="white",
                bd=0,
                padx=15,
                pady=8,
                relief="ridge",
                cursor="hand2"
            ).pack(side="left", padx=5)

        # Content frame
        self.content_frame = tk.Frame(self, bg="white", relief="groove", bd=2)
        self.content_frame.pack(fill="both", expand=True, padx=20, pady=10)

    def clear_content(self):
        for widget in self.content_frame.winfo_children():
            widget.destroy()

    def show_dashboard(self):
        self.clear_content()

        # --- Dashboard Card Frame ---
        card = tk.Frame(self.content_frame, bg="#f5f7fa", bd=2, relief="ridge")
        card.pack(padx=40, pady=30, fill="x")

        # --- Header Row ---
        header_frame = tk.Frame(card, bg="#00796b")
        header_frame.pack(fill="x")
        tk.Label(
            header_frame,
            text="👩‍🏫 Teacher Dashboard",
            font=("Segoe UI", 20, "bold"),
            bg="#00796b",
            fg="white",
            pady=18
        ).pack(fill="x")

        # --- Info Row ---
        info_frame = tk.Frame(card, bg="#f5f7fa")
        info_frame.pack(fill="x", padx=20, pady=(20, 10))

        info_frame.columnconfigure(0, weight=1)
        info_frame.columnconfigure(1, weight=3)

        # Left Column: Profile Icon
        profile_frame = tk.Frame(info_frame, bg="#f5f7fa")
        profile_frame.grid(row=0, column=0, sticky="nw")
        tk.Label(
            profile_frame,
            text="🧑‍🏫",
            font=("Segoe UI Emoji", 60),
            bg="#f5f7fa"
        ).pack(anchor="center")

        # Right Column: Teacher Data
        data_frame = tk.Frame(info_frame, bg="#f5f7fa")
        data_frame.grid(row=0, column=1, sticky="nw")

        username = self.controller.current_user
        teacher_data = self.controller.users["teachers"].get(username, {})

        def info_row(icon, label, value, fg="#333"):
            row = tk.Frame(data_frame, bg="#f5f7fa")
            row.pack(anchor="w", pady=2)
            tk.Label(row, text=icon, font=("Segoe UI Emoji", 14), bg="#f5f7fa").pack(side="left", padx=(0, 8))
            tk.Label(row, text=label, font=("Segoe UI", 11, "bold"), bg="#f5f7fa", fg="#555").pack(side="left")
            tk.Label(row, text=value, font=("Segoe UI", 11), bg="#f5f7fa", fg=fg).pack(side="left", padx=(5, 0))

        # Data rows
        full_name = f"{teacher_data.get('first_name', '')} {teacher_data.get('last_name', '')}".strip() or "N/A"
        info_row("👤", "Name:", full_name)
        info_row("📧", "Email:", teacher_data.get("email", "N/A"))
        info_row("📱", "Phone:", teacher_data.get("phone", "N/A"))
        info_row("🏢", "Department:", teacher_data.get("department", "N/A"))
        info_row("📖", "Subjects:", teacher_data.get("subjects", "N/A"))
        info_row("🎓", "Experience:", f"{teacher_data.get('years_of_experience', 'N/A')} years")

        # --- Divider ---
        divider = tk.Frame(card, bg="#e0e0e0", height=2)
        divider.pack(fill="x", pady=18)

        # --- Quick Stats Row ---
        stats_frame = tk.Frame(card, bg="#f5f7fa")
        stats_frame.pack(fill="x", padx=10, pady=(0, 10))

        stat_style = {"font": ("Segoe UI", 12, "bold"), "bg": "#f5f7fa", "fg": "#00796b"}
        tk.Label(stats_frame, text="📅 Date:", **stat_style).grid(row=0, column=0, sticky="w", padx=10)
        tk.Label(stats_frame, text=datetime.date.today().strftime("%B %d, %Y"), font=("Segoe UI", 12), bg="#f5f7fa", fg="#333").grid(row=0, column=1, sticky="w", padx=5)
        tk.Label(stats_frame, text="⏰ Login Time:", **stat_style).grid(row=0, column=2, sticky="w", padx=30)
        tk.Label(stats_frame, text=self.controller.login_time, font=("Segoe UI", 12), bg="#f5f7fa", fg="#333").grid(row=0, column=3, sticky="w", padx=5)

        # --- Welcome Message ---
        tk.Label(
            card,
            text=f"Welcome to your dashboard, {full_name if full_name != 'N/A' else 'teacher'}!",
            font=("Segoe UI", 12, "italic"),
            bg="#f5f7fa",
            fg="#00796b"
        ).pack(pady=(10, 5))

        # --- If no teacher data ---
        if not isinstance(teacher_data, dict) or not teacher_data.get("first_name"):
            tk.Label(
                card,
                text="⚠️ No teacher data available.",
                font=("Segoe UI", 12, "bold"),
                bg="#f5f7fa",
                fg="red"
            ).pack(pady=10)

    def show_menu(self):
        self.clear_content()
        # --- Menu Card Frame ---
        card = tk.Frame(self.content_frame, bg="#e0f7fa", bd=2, relief="ridge")
        card.pack(padx=60, pady=40, fill="x")

        # --- Menu Header ---
        tk.Label(
            card,
            text="📁 Teacher Menu Options",
            font=("Segoe UI", 18, "bold"),
            bg="#00897b",
            fg="white",
            pady=18
        ).pack(fill="x", pady=(0, 20))

        # --- Menu Buttons Grid ---
        menu_frame = tk.Frame(card, bg="#e0f7fa")
        menu_frame.pack(pady=10, padx=30, fill="x")

        options = [
            ("📋 Student List", self.show_student_list, "#039be5"),
            ("📅 Scheduled Classes", self.show_schedule, "#43a047"),
            ("📝 Assignment Grades", self.show_grades, "#fbc02d"),
            ("📄 Generate Report", self.generate_report, "#8e24aa"),
            ("🔍 Search Student", self.search_student, "#0288d1"),
        ]

        for i, (text, cmd, color) in enumerate(options):
            btn = tk.Button(
                menu_frame,
                text=text,
                command=cmd,
                font=("Segoe UI", 12, "bold"),
                bg=color,
                fg="white",
                activebackground="#00695c",
                activeforeground="white",
                relief="ridge",
                bd=0,
                padx=30,
                pady=18,
                cursor="hand2",
                highlightthickness=2,
                highlightbackground="#b2dfdb"
            )
            btn.grid(row=i // 2, column=i % 2, padx=25, pady=15, sticky="ew")

        # Make columns expand equally
        menu_frame.grid_columnconfigure(0, weight=1)
        menu_frame.grid_columnconfigure(1, weight=1)

        # --- Decorative Divider ---
        divider = tk.Frame(card, bg="#b2dfdb", height=2)
        divider.pack(fill="x", pady=25)

        # --- Tip or Footer ---
        tk.Label(
            card,
            text="Tip: Use the menu to quickly access all teacher features.",
            font=("Segoe UI", 11, "italic"),
            bg="#e0f7fa",
            fg="#00897b"
        ).pack(pady=(0, 10))

    def show_news(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📰 School News",
            font=("Segoe UI", 18, "bold"),
            bg="white",
            fg="#1565c0",
            pady=10
        ).pack(pady=(10, 5))

        # News items: (title, details)
        news_items = [
            ("📢 Enrollment for next semester starts May 1, 2025.", "Enrollment for the next semester will begin on May 1, 2025. Please prepare your documents and check the requirements on the school portal."),
            ("🎓 Graduation on June 15, 2025.", "The graduation ceremony will be held on June 15, 2025, at the main auditorium. Graduating students must confirm attendance by May 30."),
            ("📚 New library materials available.", "The school library has received new books and digital resources. Visit the library or check the online catalog for more info."),
            ("🏆 Sports week: April 30, 2025.", "Join the annual sports week! Events start on April 30, 2025. Register your team with the PE department."),
            ("💻 Online classes resume April 25, 2025.", "All online classes will resume on April 25, 2025. Check your schedule and ensure your devices are ready.")
        ]

        def show_details(title, details):
            messagebox.showinfo(title, details)

        for idx, (title, details) in enumerate(news_items):
            btn = tk.Button(
                self.content_frame,
                text=title,
                font=("Segoe UI", 12, "bold"),
                bg="#e3f2fd",
                fg="#1565c0",
                activebackground="#bbdefb",
                activeforeground="#0d47a1",
                relief="groove",
                bd=1,
                anchor="w",
                padx=15,
                pady=8,
                cursor="hand2",
                command=lambda t=title, d=details: show_details(t, d)
            )
            btn.pack(fill="x", padx=30, pady=6)

    def show_events(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📅 Upcoming Teacher Events",
            font=("Segoe UI", 18, "bold"),
            bg="white",
            fg="#388e3c",
            pady=10
        ).pack(pady=(10, 5))

        events = [
            ("🎉 Teacher's Day – May 5, 2025", "A special celebration to honor all teachers. Join us for awards, games, and a luncheon."),
            ("👪 Parent-Teacher Meet – May 10, 2025", "Meet with parents to discuss student progress and classroom activities."),
            ("🏆 Sports Day – May 20, 2025", "Participate or cheer in the annual sports day. Teachers vs. students events included!"),
            ("🎭 Cultural Fest – June 1, 2025", "Showcase your talents or support your students in music, dance, and art."),
            ("📖 Workshop on Teaching – June 10, 2025", "Professional development workshop on innovative teaching strategies.")
        ]

        def show_event_details(title, details):
            messagebox.showinfo(title, details)

        for idx, (title, details) in enumerate(events):
            btn = tk.Button(
                self.content_frame,
                text=title,
                font=("Segoe UI", 12, "bold"),
                bg="#e8f5e9",
                fg="#388e3c",
                activebackground="#c8e6c9",
                activeforeground="#1b5e20",
                relief="groove",
                bd=2,
                anchor="w",
                padx=15,
                pady=8,
                cursor="hand2",
                command=lambda t=title, d=details: show_event_details(t, d)
            )
            btn.pack(fill="x", padx=40, pady=6)

    def show_lesson_plan(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📚 Lesson Plan",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=20, pady=10)

        columns = ("Date", "Subject", "Topic", "Description")
        self.lesson_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=10)
        self.lesson_table.pack(fill="both", expand=True, padx=10, pady=10)

        for col in columns:
            self.lesson_table.heading(col, text=col)
            self.lesson_table.column(col, anchor="center")

        # In-memory lesson plan data
        if not hasattr(self.controller, "lesson_plan_data"):
            self.controller.lesson_plan_data = [
                ("2025-04-25", "Math", "Algebra", "Intro to linear equations"),
                ("2025-04-26", "Science", "Physics", "Newton's Laws"),
                ("2025-04-27", "English", "Grammar", "Parts of Speech"),
                ("2025-04-28", "History", "WWII", "Causes and impact"),
                ("2025-04-29", "CompSci", "Python", "Basics of syntax")
            ]
        for row in self.controller.lesson_plan_data:
            self.lesson_table.insert("", "end", values=row)

        # Button frame for actions
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=10)

        tk.Button(
            button_frame, text="Add", command=self.add_lesson,
            bg="#5cb85c", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)
        tk.Button(
            button_frame, text="Edit", command=self.edit_lesson,
            bg="#5bc0de", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)
        tk.Button(
            button_frame, text="Delete", command=self.delete_lesson,
            bg="#d9534f", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)
        tk.Button(
            button_frame, text="Print", command=self.print_lesson_plan,
            bg="#388e3c", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)
        tk.Button(
            button_frame, text="Exit", command=self.show_menu,
            bg="#d32f2f", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)

    def add_lesson(self):
        def save():
            date = date_entry.get().strip()
            subject = subject_entry.get().strip()
            topic = topic_entry.get().strip()
            desc = desc_entry.get().strip()
            if date and subject and topic and desc:
                self.controller.lesson_plan_data.append((date, subject, topic, desc))
                self.lesson_table.insert("", "end", values=(date, subject, topic, desc))
                top.destroy()
            else:
                messagebox.showerror("Error", "All fields are required.")

        top = tk.Toplevel(self.master)
        top.title("Add Lesson")
        top.geometry("400x300")
        top.configure(bg="#eef6fc")
        top.resizable(False, False)

        form = tk.Frame(top, bg="#eef6fc")
        form.pack(pady=20)
        tk.Label(form, text="Date:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=0, column=0, sticky="w", pady=5, padx=10)
        date_entry = tk.Entry(form, font=("Segoe UI", 11))
        date_entry.grid(row=0, column=1, pady=5, padx=10)
        tk.Label(form, text="Subject:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=1, column=0, sticky="w", pady=5, padx=10)
        subject_entry = tk.Entry(form, font=("Segoe UI", 11))
        subject_entry.grid(row=1, column=1, pady=5, padx=10)
        tk.Label(form, text="Topic:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=2, column=0, sticky="w", pady=5, padx=10)
        topic_entry = tk.Entry(form, font=("Segoe UI", 11))
        topic_entry.grid(row=2, column=1, pady=5, padx=10)
        tk.Label(form, text="Description:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=3, column=0, sticky="w", pady=5, padx=10)
        desc_entry = tk.Entry(form, font=("Segoe UI", 11))
        desc_entry.grid(row=3, column=1, pady=5, padx=10)
        tk.Button(top, text="Save", command=save, bg="#5cb85c", fg="white", font=("Segoe UI", 11, "bold")).pack(pady=15)

    def edit_lesson(self):
        selected = self.lesson_table.selection()
        if not selected:
            messagebox.showerror("Error", "Please select a lesson to edit.")
            return
        values = self.lesson_table.item(selected, "values")

        def save_edit():
            date = date_entry.get().strip()
            subject = subject_entry.get().strip()
            topic = topic_entry.get().strip()
            desc = desc_entry.get().strip()
            if date and subject and topic and desc:
                idx = self.lesson_table.index(selected)
                self.controller.lesson_plan_data[idx] = (date, subject, topic, desc)
                self.lesson_table.item(selected, values=(date, subject, topic, desc))
                top.destroy()
            else:
                messagebox.showerror("Error", "All fields are required.")

        top = tk.Toplevel(self.master)
        top.title("Edit Lesson")
        top.geometry("400x300")
        top.configure(bg="#eef6fc")
        top.resizable(False, False)

        form = tk.Frame(top, bg="#eef6fc")
        form.pack(pady=20)
        tk.Label(form, text="Date:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=0, column=0, sticky="w", pady=5, padx=10)
        date_entry = tk.Entry(form, font=("Segoe UI", 11))
        date_entry.insert(0, values[0])
        date_entry.grid(row=0, column=1, pady=5, padx=10)
        tk.Label(form, text="Subject:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=1, column=0, sticky="w", pady=5, padx=10)
        subject_entry = tk.Entry(form, font=("Segoe UI", 11))
        subject_entry.insert(0, values[1])
        subject_entry.grid(row=1, column=1, pady=5, padx=10)
        tk.Label(form, text="Topic:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=2, column=0, sticky="w", pady=5, padx=10)
        topic_entry = tk.Entry(form, font=("Segoe UI", 11))
        topic_entry.insert(0, values[2])
        topic_entry.grid(row=2, column=1, pady=5, padx=10)
        tk.Label(form, text="Description:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=3, column=0, sticky="w", pady=5, padx=10)
        desc_entry = tk.Entry(form, font=("Segoe UI", 11))
        desc_entry.insert(0, values[3])
        desc_entry.grid(row=3, column=1, pady=5, padx=10)
        tk.Button(top, text="Save Changes", command=save_edit, bg="#5bc0de", fg="white", font=("Segoe UI", 11, "bold")).pack(pady=15)

    def delete_lesson(self):
        selected = self.lesson_table.selection()
        if not selected:
            messagebox.showerror("Error", "Please select a lesson to delete.")
            return
        idx = self.lesson_table.index(selected)
        self.lesson_table.delete(selected)
        del self.controller.lesson_plan_data[idx]
        messagebox.showinfo("Deleted", "Lesson deleted successfully.")

    def print_lesson_plan(self):
        if not self.lesson_table.get_children():
            messagebox.showwarning("No Data", "No lesson plan data to print!")
        else:
            messagebox.showinfo("Print", "Lesson plan printed successfully!")
        

    def show_student_list(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="Registered Students",
            font=("Segoe UI", 18, "bold"),
            bg="white",
            fg="#4682B4",
            pady=10
        ).pack(pady=10)

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=30, pady=10)

        columns = ("Username", "Program")
        student_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        student_table.pack(fill="both", expand=True)

        for col in columns:
            student_table.heading(col, text=col)
            student_table.column(col, anchor="center", width=200)

        # Get registered students and their program
        students = self.controller.users.get("students", {})
        for username, info in students.items():
            # Only show if info is a dict (i.e., registered student)
            if isinstance(info, dict):
                # Try to get program from enrollments, else from registration, else "N/A"
                program = "N/A"
                if hasattr(self.controller, "enrollments"):
                    enrolls = self.controller.enrollments.get(username, [])
                    if enrolls:
                        program = enrolls[-1].get("program", "N/A")
                if program == "N/A":
                    program = info.get("program", info.get("course", "N/A"))
                student_table.insert("", "end", values=(username, program))

        # Button frame below the table
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=10)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            bg="#d9534f",
            fg="white",
            font=("Segoe UI", 11, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack()
            
    def show_schedule(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="Scheduled Classes",
            font=("Segoe UI", 18, "bold"),
            bg="white",
            fg="#4682B4",
            pady=10
        ).pack(pady=10)

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=20, pady=10)

        columns = ("Date", "Time", "Subject", "Teacher")
        self.schedule_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=12)
        self.schedule_table.pack(fill="both", expand=True)

        for col in columns:
            self.schedule_table.heading(col, text=col)
            self.schedule_table.column(col, anchor="center", width=120)

        # Use in-memory schedule storage
        if not hasattr(self.controller, "schedule_data"):
            self.controller.schedule_data = [
                ("2025-04-26", "9:00 AM", "Mathematics", "Mr. Johnson"),
                ("2025-04-26", "11:00 AM", "Science", "Ms. Lee"),
                ("2025-04-27", "1:00 PM", "History", "Mr. Smith"),
                ("2025-04-27", "3:00 PM", "English", "Mrs. Davis")
            ]
        for row in self.controller.schedule_data:
            self.schedule_table.insert("", "end", values=row)

        # Button frame
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=15)

        tk.Button(
            button_frame, text="Add", command=self.add_schedule,
            bg="#5cb85c", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)
        tk.Button(
            button_frame, text="Edit", command=self.edit_schedule,
            bg="#5bc0de", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)
        tk.Button(
            button_frame, text="Delete", command=self.delete_schedule,
            bg="#d9534f", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)
        tk.Button(
            button_frame, text="Exit", command=self.show_menu,
            bg="#d32f2f", fg="white", font=("Segoe UI", 11, "bold"),
            width=12, relief="ridge", bd=0, padx=10, pady=6, cursor="hand2"
        ).pack(side="left", padx=8)

    def add_schedule(self):
        def save():
            date = date_entry.get().strip()
            time = time_entry.get().strip()
            subject = subject_entry.get().strip()
            teacher = teacher_entry.get().strip()
            if date and time and subject and teacher:
                self.controller.schedule_data.append((date, time, subject, teacher))
                self.schedule_table.insert("", "end", values=(date, time, subject, teacher))
                top.destroy()
            else:
                messagebox.showerror("Error", "All fields are required.")

        top = tk.Toplevel(self.master)
        top.title("Add Scheduled Class")
        top.geometry("350x250")
        top.configure(bg="#eef6fc")
        top.resizable(False, False)

        form = tk.Frame(top, bg="#eef6fc")
        form.pack(pady=20)
        tk.Label(form, text="Date:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=0, column=0, sticky="w", pady=5, padx=10)
        date_entry = tk.Entry(form, font=("Segoe UI", 11))
        date_entry.grid(row=0, column=1, pady=5, padx=10)
        tk.Label(form, text="Time:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=1, column=0, sticky="w", pady=5, padx=10)
        time_entry = tk.Entry(form, font=("Segoe UI", 11))
        time_entry.grid(row=1, column=1, pady=5, padx=10)
        tk.Label(form, text="Subject:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=2, column=0, sticky="w", pady=5, padx=10)
        subject_entry = tk.Entry(form, font=("Segoe UI", 11))
        subject_entry.grid(row=2, column=1, pady=5, padx=10)
        tk.Label(form, text="Teacher:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=3, column=0, sticky="w", pady=5, padx=10)
        teacher_entry = tk.Entry(form, font=("Segoe UI", 11))
        teacher_entry.grid(row=3, column=1, pady=5, padx=10)
        tk.Button(top, text="Save", command=save, bg="#5cb85c", fg="white", font=("Segoe UI", 11, "bold")).pack(pady=15)

    def edit_schedule(self):
        selected = self.schedule_table.selection()
        if not selected:
            messagebox.showerror("Error", "Please select a schedule to edit.")
            return
        values = self.schedule_table.item(selected, "values")

        def save_edit():
            date = date_entry.get().strip()
            time = time_entry.get().strip()
            subject = subject_entry.get().strip()
            teacher = teacher_entry.get().strip()
            if date and time and subject and teacher:
                idx = self.schedule_table.index(selected)
                self.controller.schedule_data[idx] = (date, time, subject, teacher)
                self.schedule_table.item(selected, values=(date, time, subject, teacher))
                top.destroy()
            else:
                messagebox.showerror("Error", "All fields are required.")

        top = tk.Toplevel(self.master)
        top.title("Edit Scheduled Class")
        top.geometry("350x250")
        top.configure(bg="#eef6fc")
        top.resizable(False, False)

        form = tk.Frame(top, bg="#eef6fc")
        form.pack(pady=20)
        tk.Label(form, text="Date:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=0, column=0, sticky="w", pady=5, padx=10)
        date_entry = tk.Entry(form, font=("Segoe UI", 11))
        date_entry.insert(0, values[0])
        date_entry.grid(row=0, column=1, pady=5, padx=10)
        tk.Label(form, text="Time:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=1, column=0, sticky="w", pady=5, padx=10)
        time_entry = tk.Entry(form, font=("Segoe UI", 11))
        time_entry.insert(0, values[1])
        time_entry.grid(row=1, column=1, pady=5, padx=10)
        tk.Label(form, text="Subject:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=2, column=0, sticky="w", pady=5, padx=10)
        subject_entry = tk.Entry(form, font=("Segoe UI", 11))
        subject_entry.insert(0, values[2])
        subject_entry.grid(row=2, column=1, pady=5, padx=10)
        tk.Label(form, text="Teacher:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=3, column=0, sticky="w", pady=5, padx=10)
        teacher_entry = tk.Entry(form, font=("Segoe UI", 11))
        teacher_entry.insert(0, values[3])
        teacher_entry.grid(row=3, column=1, pady=5, padx=10)
        tk.Button(top, text="Save Changes", command=save_edit, bg="#5bc0de", fg="white", font=("Segoe UI", 11, "bold")).pack(pady=15)

    def delete_schedule(self):
        selected = self.schedule_table.selection()
        if not selected:
            messagebox.showerror("Error", "Please select a schedule to delete.")
            return
        idx = self.schedule_table.index(selected)
        self.schedule_table.delete(selected)
        del self.controller.schedule_data[idx]
        messagebox.showinfo("Deleted", "Schedule deleted successfully.")

# ...existing code...
        
    def show_grades(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="Assignment Grades",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # Button frame (place above the table, horizontal layout)
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=(0, 10), anchor="w")

        tk.Button(
            button_frame,
            text="Add Grade",
            command=lambda: self.add_grade(self.grades_table),
            bg="#5cb85c",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack(side="left", padx=5)

        tk.Button(
            button_frame,
            text="Edit Grade",
            command=lambda: self.edit_grade(self.grades_table),
            bg="#5bc0de",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack(side="left", padx=5)

        tk.Button(
            button_frame,
            text="Delete Grade",
            command=lambda: self.delete_grade(self.grades_table),
            bg="#d9534f",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack(side="left", padx=5)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            bg="#d32f2f",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack(side="left", padx=5)

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=20, pady=10)

        columns = ("Student Name", "Assignment", "Grade")
        self.grades_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        self.grades_table.pack(fill="both", expand=True)

        for col in columns:
            self.grades_table.heading(col, text=col)
            self.grades_table.column(col, width=200, anchor="center")

        # Load grades from shared dictionary
        for student, assignments in self.controller.grades.items():
            for entry in assignments:
                self.grades_table.insert("", "end", values=(student, entry["assignment"], entry["grade"]))

    def add_grade(self, table):
        def save_new():
            student = student_entry.get().strip()
            assignment = assignment_entry.get().strip()
            grade = grade_entry.get().strip()
            if student and assignment and grade:
                # Get instructor name (current logged-in teacher)
                instructor_username = self.controller.current_user
                # Try to get full name, else use username
                teacher_info = self.controller.users["teachers"].get(instructor_username, {})
                instructor_name = f"{teacher_info.get('first_name', '')} {teacher_info.get('last_name', '')}".strip() or instructor_username

                # Update shared grades dictionary
                if student not in self.controller.grades:
                    self.controller.grades[student] = []
                self.controller.grades[student].append({
                    "assignment": assignment,
                    "grade": grade,
                    "instructor": instructor_name
                })
                table.insert("", "end", values=(student, assignment, grade, instructor_name))
                top.destroy()
            else:
                messagebox.showerror("Error", "All fields are required.")

        top = tk.Toplevel(self.master)
        top.title("Add Grade")
        top.geometry("400x300")
        top.configure(bg="#eef6fc")
        top.resizable(False, False)

        form_frame = tk.Frame(top, bg="#eef6fc")
        form_frame.pack(pady=20)

        tk.Label(form_frame, text="Student Name:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=0, column=0, sticky="w", pady=5, padx=10)
        student_entry = tk.Entry(form_frame, font=("Segoe UI", 11))
        student_entry.grid(row=0, column=1, pady=5, padx=10)

        tk.Label(form_frame, text="Assignment:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=1, column=0, sticky="w", pady=5, padx=10)
        assignment_entry = tk.Entry(form_frame, font=("Segoe UI", 11))
        assignment_entry.grid(row=1, column=1, pady=5, padx=10)

        tk.Label(form_frame, text="Grade:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=2, column=0, sticky="w", pady=5, padx=10)
        grade_entry = tk.Entry(form_frame, font=("Segoe UI", 11))
        grade_entry.grid(row=2, column=1, pady=5, padx=10)

        tk.Button(top, text="Save Grade", command=save_new, bg="#5cb85c", fg="white", font=("Segoe UI", 11, "bold")).pack(pady=15)
        
    def edit_grade(self, table):
        selected = table.selection()
        if not selected:
            messagebox.showerror("Error", "Please select a grade to edit.")
            return
        item = table.item(selected)
        values = item['values']

        def save_edit():
            new_student = student_entry.get().strip()
            new_assignment = assignment_entry.get().strip()
            new_grade = grade_entry.get().strip()
            if new_student and new_assignment and new_grade:
                # Update shared grades dictionary
                # Remove old entry
                old_student, old_assignment = values[0], values[1]
                if old_student in self.controller.grades:
                    self.controller.grades[old_student] = [
                        g for g in self.controller.grades[old_student]
                        if g["assignment"] != old_assignment
                    ]
                # Add new entry
                if new_student not in self.controller.grades:
                    self.controller.grades[new_student] = []
                self.controller.grades[new_student].append({"assignment": new_assignment, "grade": new_grade})
                table.item(selected, values=(new_student, new_assignment, new_grade))
                top.destroy()
            else:
                messagebox.showerror("Error", "All fields are required.")

        top = tk.Toplevel(self.master)
        top.title("Edit Grade")
        top.geometry("350x250")
        top.configure(bg="#eef6fc")
        top.resizable(False, False)

        form_frame = tk.Frame(top, bg="#eef6fc")
        form_frame.pack(pady=20)

        tk.Label(form_frame, text="Student Name:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=0, column=0, sticky="w", pady=5, padx=10)
        student_entry = tk.Entry(form_frame, font=("Segoe UI", 11))
        student_entry.insert(0, values[0])
        student_entry.grid(row=0, column=1, pady=5, padx=10)

        tk.Label(form_frame, text="Assignment:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=1, column=0, sticky="w", pady=5, padx=10)
        assignment_entry = tk.Entry(form_frame, font=("Segoe UI", 11))
        assignment_entry.insert(0, values[1])
        assignment_entry.grid(row=1, column=1, pady=5, padx=10)

        tk.Label(form_frame, text="Grade:", bg="#eef6fc", font=("Segoe UI", 11)).grid(row=2, column=0, sticky="w", pady=5, padx=10)
        grade_entry = tk.Entry(form_frame, font=("Segoe UI", 11))
        grade_entry.insert(0, values[2])
        grade_entry.grid(row=2, column=1, pady=5, padx=10)

        tk.Button(top, text="Save Changes", command=save_edit, bg="#5bc0de", fg="white", font=("Segoe UI", 11, "bold")).pack(pady=15)

    def delete_grade(self, table):
        selected_item = table.selection()
        if selected_item:
            values = table.item(selected_item)['values']
            student, assignment = values[0], values[1]
            # Remove from shared grades dictionary
            if student in self.controller.grades:
                self.controller.grades[student] = [
                    g for g in self.controller.grades[student]
                    if g["assignment"] != assignment
                ]
            table.delete(selected_item)
            messagebox.showinfo("Deleted", "Grade deleted successfully.")
        else:
            messagebox.showerror("Error", "Please select a grade to delete.")
            
    # ...existing code...
    def generate_report(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📄 Generate Student Report",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # --- Button frame (now above the search and table, horizontal layout) ---
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=(0, 10), anchor="w")

        def print_report():
            if not student_table.get_children():
                messagebox.showwarning("No Data", "No student data to print!")
            else:
                messagebox.showinfo("Print", "Report printed successfully!")

        tk.Button(
            button_frame,
            text="Print",
            command=print_report,
            bg="#4682B4",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack(side="left", padx=10)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            bg="#d9534f",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack(side="left", padx=10)

        # --- Search frame ---
        search_frame = tk.Frame(self.content_frame, bg="white")
        search_frame.pack(pady=10)

        tk.Label(
            search_frame,
            text="Search Student:",
            font=("Segoe UI", 12, "bold"),
            bg="white",
            fg="#333"
        ).grid(row=0, column=0, padx=10)

        search_var = tk.StringVar()
        search_entry = tk.Entry(search_frame, textvariable=search_var, font=("Segoe UI", 12))
        search_entry.grid(row=0, column=1, padx=10)

        # Table frame for student report
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=20, pady=20)

        columns = ("Attribute", "Details")
        student_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        student_table.pack(fill="both", expand=True)

        for col in columns:
            student_table.heading(col, text=col)
            student_table.column(col, width=400, anchor="center")

        def search_student(name):
            # Clear the table
            for item in student_table.get_children():
                student_table.delete(item)
            # Get student info from registered students
            student_info = self.controller.users["students"].get(name)
            if student_info and isinstance(student_info, dict):
                student_id = student_info.get("id", "N/A")
                course = student_info.get("course", "N/A")
                year_level = student_info.get("year_level", "N/A")
                email = student_info.get("email", "N/A")
                phone = student_info.get("phone", "N/A")
                full_name = f"{student_info.get('first_name', '')} {student_info.get('last_name', '')}".strip()
                student_table.insert("", "end", values=("Name", full_name))
                student_table.insert("", "end", values=("Student ID", student_id))
                student_table.insert("", "end", values=("Course", course))
                student_table.insert("", "end", values=("Year Level", year_level))
                student_table.insert("", "end", values=("Email", email))
                student_table.insert("", "end", values=("Phone", phone))
                # Show grades if available
                grades = self.controller.grades.get(name, [])
                student_table.insert("", "end", values=("Grades", ""))
                for entry in grades:
                    student_table.insert("", "end", values=(f"   {entry['assignment']}", entry['grade']))
            else:
                messagebox.showinfo("Not Found", f"Student '{name}' not found!")

        tk.Button(
            search_frame,
            text="Search",
            command=lambda: search_student(search_var.get()),
            bg="#4682B4",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=10,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).grid(row=0, column=2, padx=10)
# ...existing code...
                
    def search_student(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="🔍 Search Student",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # Search frame
        search_frame = tk.Frame(self.content_frame, bg="white")
        search_frame.pack(pady=10)

        tk.Label(
            search_frame,
            text="Enter Student Username:",
            font=("Segoe UI", 12, "bold"),
            bg="white",
            fg="#333"
        ).grid(row=0, column=0, padx=10)

        search_var = tk.StringVar()
        search_entry = tk.Entry(search_frame, textvariable=search_var, font=("Segoe UI", 12))
        search_entry.grid(row=0, column=1, padx=10)

        def do_search():
            # Clear the table
            for item in student_table.get_children():
                student_table.delete(item)
            # Search in registered students
            username = search_var.get().strip()
            student_info = self.controller.users["students"].get(username)
            if student_info and isinstance(student_info, dict):
                full_name = f"{student_info.get('first_name', '')} {student_info.get('last_name', '')}".strip()
                student_id = student_info.get("id", "N/A")
                course = student_info.get("course", "N/A")
                year_level = student_info.get("year_level", "N/A")
                email = student_info.get("email", "N/A")
                phone = student_info.get("phone", "N/A")
                student_table.insert("", "end", values=("Name", full_name))
                student_table.insert("", "end", values=("Student ID", student_id))
                student_table.insert("", "end", values=("Course", course))
                student_table.insert("", "end", values=("Year Level", year_level))
                student_table.insert("", "end", values=("Email", email))
                student_table.insert("", "end", values=("Phone", phone))
            else:
                messagebox.showinfo("Not Found", f"Student '{username}' not found or not registered.")

        tk.Button(
            search_frame,
            text="Search",
            command=do_search,
            bg="#4682B4",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=10,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).grid(row=0, column=2, padx=10)

        # Table frame for student info
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=20, pady=20)

        columns = ("Attribute", "Details")
        student_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=10)
        student_table.pack(fill="both", expand=True)

        for col in columns:
            student_table.heading(col, text=col)
            student_table.column(col, width=400, anchor="center")

        # Button frame for Exit
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=10)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            bg="#d9534f",
            fg="white",
            font=("Segoe UI", 10, "bold"),
            width=15,
            relief="ridge",
            bd=0,
            padx=10,
            pady=6,
            cursor="hand2"
        ).pack()

    def back_to_login(self):
        if messagebox.askyesno("Logout", "Are you sure you want to logout?"):
            self.pack_forget()
            self.destroy()  # Ensure the frame is fully destroyed
            self.controller.frames.pop(TeacherPortal, None)  # Remove from controller
            login_frame = self.controller.frames[LoginPage]
            login_frame.username_entry.delete(0, tk.END)
            login_frame.password_entry.delete(0, tk.END)
            login_frame.pack(fill="both", expand=True)
            self.controller.show_frame(LoginPage)


class StudentPortal(tk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent, bg="#e0f7fa")
        self.controller = controller

        # Header
        tk.Label(
            self,
            text="🎓 KLL Student Management System",
            font=("Segoe UI", 24, "bold"),
            bg="#0288d1",
            fg="white",
            pady=20
        ).pack(fill="x")

        # Top Navigation Buttons
        button_frame = tk.Frame(self, bg="#e0f7fa")
        button_frame.pack(fill="x", pady=10)

        buttons = [
            ("📊 DASHBOARD", self.show_dashboard),
            ("📁 MENU", self.show_menu),
            ("📰 NEWS", self.show_news),
            ("📅 EVENTS", self.show_events),
            ("❌ LOGOUT", self.back_to_login)
        ]

        for text, command in buttons:
            bg_color = "#0288d1" if "LOGOUT" not in text else "#d32f2f"
            tk.Button(
                button_frame,
                text=text,
                command=command,
                bg=bg_color,
                fg="white",
                font=("Segoe UI", 10, "bold"),
                padx=15,
                pady=8,
                relief="ridge",
                bd=0,
                activebackground="#0277bd" if "LOGOUT" not in text else "#b71c1c",
                cursor="hand2"
            ).pack(side="left", padx=5)

        # Content Frame Below
        self.content_frame = tk.Frame(self, bg="white", relief="groove", bd=2)
        self.content_frame.pack(fill="both", expand=True, padx=20, pady=10)

    def clear_content(self):
        for widget in self.content_frame.winfo_children():
            widget.destroy()

    def show_dashboard(self):
        self.clear_content()

        # --- Dashboard Card Frame ---
        card = tk.Frame(self.content_frame, bg="#f5fafd", bd=2, relief="ridge")
        card.pack(padx=40, pady=30, fill="x")

        # --- Header Row ---
        header_frame = tk.Frame(card, bg="#0288d1")
        header_frame.pack(fill="x")
        tk.Label(
            header_frame,
            text="🎓 Student Dashboard",
            font=("Segoe UI", 20, "bold"),
            bg="#0288d1",
            fg="white",
            pady=18
        ).pack(fill="x")

        # --- Info Row ---
        info_frame = tk.Frame(card, bg="#f5fafd")
        info_frame.pack(fill="x", padx=20, pady=(20, 10))

        info_frame.columnconfigure(0, weight=1)
        info_frame.columnconfigure(1, weight=3)

        # Left Column: Profile Icon
        profile_frame = tk.Frame(info_frame, bg="#f5fafd")
        profile_frame.grid(row=0, column=0, sticky="nw")
        tk.Label(
            profile_frame,
            text="🧑‍🎓",
            font=("Segoe UI Emoji", 60),
            bg="#f5fafd"
        ).pack(anchor="center")

        # Right Column: Student Data
        data_frame = tk.Frame(info_frame, bg="#f5fafd")
        data_frame.grid(row=0, column=1, sticky="nw")

        username = self.controller.current_user
        student_data = self.controller.users["students"].get(username, {})

        def info_row(icon, label, value, fg="#333"):
            row = tk.Frame(data_frame, bg="#f5fafd")
            row.pack(anchor="w", pady=2)
            tk.Label(row, text=icon, font=("Segoe UI Emoji", 14), bg="#f5fafd").pack(side="left", padx=(0, 8))
            tk.Label(row, text=label, font=("Segoe UI", 11, "bold"), bg="#f5fafd", fg="#555").pack(side="left")
            tk.Label(row, text=value, font=("Segoe UI", 11), bg="#f5fafd", fg=fg).pack(side="left", padx=(5, 0))

        # Data rows
        full_name = f"{student_data.get('first_name', '')} {student_data.get('last_name', '')}".strip() or "N/A"
        info_row("👤", "Name:", full_name)
        info_row("📧", "Email:", student_data.get("email", "N/A"))
        info_row("📱", "Phone:", student_data.get("phone", "N/A"))
        info_row("🎒", "Grade Level:", student_data.get("grade_level", student_data.get("year_level", "N/A")))
        info_row("📚", "Course:", student_data.get("course", student_data.get("program", "N/A")))

        # --- Divider ---
        divider = tk.Frame(card, bg="#e0e0e0", height=2)
        divider.pack(fill="x", pady=18)

        # --- Quick Stats Row ---
        stats_frame = tk.Frame(card, bg="#f5fafd")
        stats_frame.pack(fill="x", padx=10, pady=(0, 10))

        stat_style = {"font": ("Segoe UI", 12, "bold"), "bg": "#f5fafd", "fg": "#0288d1"}
        tk.Label(stats_frame, text="📅 Date:", **stat_style).grid(row=0, column=0, sticky="w", padx=10)
        tk.Label(stats_frame, text=datetime.date.today().strftime("%B %d, %Y"), font=("Segoe UI", 12), bg="#f5fafd", fg="#333").grid(row=0, column=1, sticky="w", padx=5)
        tk.Label(stats_frame, text="⏰ Login Time:", **stat_style).grid(row=0, column=2, sticky="w", padx=30)
        tk.Label(stats_frame, text=self.controller.login_time, font=("Segoe UI", 12), bg="#f5fafd", fg="#333").grid(row=0, column=3, sticky="w", padx=5)

        # --- Enrolled Courses ---
        enrolled_courses = []
        if hasattr(self.controller, "enrollments"):
            user_enrollments = self.controller.enrollments.get(username, [])
            for row in user_enrollments:
                enrolled_courses.append(f"{row['course_code']} - {row['course_name']} ({row['semester']})")

        if enrolled_courses:
            tk.Label(card, text="📚 Enrolled Courses:", font=("Segoe UI", 12, "bold"), bg="#f5fafd", fg="#0288d1").pack(anchor="w", padx=30, pady=(10, 0))
            for course in enrolled_courses:
                tk.Label(card, text=course, font=("Segoe UI", 12), bg="#f5fafd").pack(anchor="w", padx=50, pady=2)
        else:
            tk.Label(card, text="📚 No enrolled courses yet.", font=("Segoe UI", 12), bg="#f5fafd", fg="gray").pack(anchor="w", padx=30, pady=10)

        # --- Welcome Message ---
        tk.Label(
            card,
            text=f"Welcome to your dashboard, {full_name if full_name != 'N/A' else 'student'}!",
            font=("Segoe UI", 12, "italic"),
            bg="#f5fafd",
            fg="#0288d1"
        ).pack(pady=(10, 5))

        # --- No Data Warning ---
        if not isinstance(student_data, dict) or not student_data.get("first_name"):
            tk.Label(
                card,
                text="⚠️ No student data found.",
                font=("Segoe UI", 12, "bold"),
                bg="#f5fafd",
                fg="red"
            ).pack(pady=10)

        
    def show_menu(self):
        self.clear_content()

        # --- Menu Card Frame ---
        card = tk.Frame(self.content_frame, bg="#e3f2fd", bd=2, relief="ridge")
        card.pack(padx=60, pady=40, fill="x")

        # --- Menu Header ---
        tk.Label(
            card,
            text="📁 Course Menu Options",
            font=("Segoe UI", 18, "bold"),
            bg="#0288d1",
            fg="white",
            pady=18
        ).pack(fill="x", pady=(0, 20))

        # --- Menu Buttons Grid ---
        menu_frame = tk.Frame(card, bg="#e3f2fd")
        menu_frame.pack(pady=10, padx=30, fill="x")

        button_data = [
            ("📝 Enroll in Courses", self.show_enrollment, "#039be5"),
            ("📅 View Schedule", self.show_schedule, "#43a047"),
            ("📈 View Grades", self.show_grades, "#fbc02d"),
            ("📚 View Courses", self.show_courses, "#8e24aa"),
            ("👨‍🏫 Instructors", self.show_instructors, "#0288d1"),
        ]

        # Create a grid with 2 columns
        rows = (len(button_data) + 1) // 2  # handles odd buttons

        for i, (text, command, color) in enumerate(button_data):
            row = i // 2
            col = i % 2

            btn = tk.Button(
                menu_frame,
                text=text,
                command=command,
                font=("Segoe UI", 12, "bold"),
                bg=color,
                fg="white",
                activebackground="#0277bd" if color == "#039be5" else "#00695c",
                activeforeground="white",
                relief="ridge",
                bd=0,
                padx=30,
                pady=18,
                cursor="hand2",
                highlightthickness=2,
                highlightbackground="#b3e5fc",
                wraplength=180,
                justify="center"
            )
            btn.grid(row=row, column=col, padx=25, pady=15, sticky="nsew")

        # Configure column weights so buttons expand evenly
        for col in range(2):
            menu_frame.grid_columnconfigure(col, weight=1)
        for row in range(rows):
            menu_frame.grid_rowconfigure(row, weight=1)

        # --- Decorative Divider ---
        divider = tk.Frame(card, bg="#b3e5fc", height=2)
        divider.pack(fill="x", pady=25)

        # --- Tip or Footer ---
        tk.Label(
            card,
            text="Tip: Use the menu to access all student features quickly.",
            font=("Segoe UI", 11, "italic"),
            bg="#e3f2fd",
            fg="#0288d1"
        ).pack(pady=(0, 10))

    def show_news(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📰 Latest News",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # News items: (title, details)
        news_items = [
            ("📢 Enrollment opens May 1, 2025.", "Enrollment for the next semester will begin on May 1, 2025. Please prepare your documents and check the requirements on the school portal."),
            ("🎓 Graduation: June 15, 2025.", "The graduation ceremony will be held on June 15, 2025, at the main auditorium. Graduating students must confirm attendance by May 30."),
            ("📚 Library updated with new materials.", "The school library has received new books and digital resources. Visit the library or check the online catalog for more info."),
            ("🏆 Sports week starts April 30, 2025.", "Join the annual sports week! Events start on April 30, 2025. Register your team with the PE department."),
            ("💻 Online classes resume April 25, 2025.", "All online classes will resume on April 25, 2025. Check your schedule and ensure your devices are ready.")
        ]

        def show_details(title, details):
            messagebox.showinfo(title, details)

        for idx, (title, details) in enumerate(news_items):
            btn = tk.Button(
                self.content_frame,
                text=title,
                font=("Segoe UI", 12, "bold"),
                bg="#e3f2fd",
                fg="#1565c0",
                activebackground="#bbdefb",
                activeforeground="#0d47a1",
                relief="groove",
                bd=1,
                anchor="w",
                padx=15,
                pady=8,
                cursor="hand2",
                command=lambda t=title, d=details: show_details(t, d)
            )
            btn.pack(fill="x", padx=30, pady=6)
            
    def show_events(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📅 Upcoming Student Events",
            font=("Segoe UI", 18, "bold"),
            bg="white",
            fg="#388e3c",
            pady=10
        ).pack(pady=(10, 5))

        events = [
            ("🎉 Welcome Day – May 5, 2025", "Kick off the new school year with fun activities and a welcome program for all students."),
            ("👨‍👩‍👧 Orientation – May 10, 2025", "Orientation for new students and parents. Meet your teachers and learn about school policies."),
            ("🏆 Sports Meet – May 20, 2025", "Annual sports meet with various competitions. Register your team with the PE department."),
            ("🎭 Cultural Fest – June 1, 2025", "A celebration of culture and talent. Participate in music, dance, and art events."),
            ("📖 Research Workshop – June 10, 2025", "Workshop on research skills and academic writing for all interested students.")
        ]

        def show_event_details(title, details):
            messagebox.showinfo(title, details)
            
        for idx, (title, details) in enumerate(events):
            btn = tk.Button(
                self.content_frame,
                text=title,
                font=("Segoe UI", 12, "bold"),
                bg="#e8f5e9",
                fg="#388e3c",
                activebackground="#c8e6c9",
                activeforeground="#1b5e20",
                relief="groove",
                bd=2,
                anchor="w",
                padx=15,
                pady=8,
                cursor="hand2",
                command=lambda t=title, d=details: show_event_details(t, d)
            )
            btn.pack(fill="x", padx=40, pady=6)
            
    def show_enrollment(self):
        self.clear_content()

        # --- Course Enrollment Form with Program Selection ---
        form_frame = tk.Frame(self.content_frame, bg="white")
        form_frame.pack(pady=30, padx=40, fill="both", expand=True)

        tk.Label(form_frame, text="Course Enrollment Form", font=("Segoe UI", 16, "bold"), bg="white").grid(row=0, column=0, columnspan=2, pady=(0, 20))

        # Program selection
        tk.Label(form_frame, text="Program:", font=("Segoe UI", 12), bg="white").grid(row=1, column=0, sticky="e", pady=6, padx=8)
        program_options = [
            "BS Computer Science",
            "BS Information Technology",
            "BS Engineering",
            "BS Business Administration",
            "BS Education",
            "BS Nursing",
            "Other"
        ]
        program = ttk.Combobox(form_frame, values=program_options, state="readonly", font=("Segoe UI", 12), width=28)
        program.grid(row=1, column=1, pady=6, padx=8)
        program.current(0)

        tk.Label(form_frame, text="Course Code:", font=("Segoe UI", 12), bg="white").grid(row=2, column=0, sticky="e", pady=6, padx=8)
        course_code = ttk.Entry(form_frame, font=("Segoe UI", 12), width=30)
        course_code.grid(row=2, column=1, pady=6, padx=8)

        tk.Label(form_frame, text="Course Name:", font=("Segoe UI", 12), bg="white").grid(row=3, column=0, sticky="e", pady=6, padx=8)
        course_name = ttk.Entry(form_frame, font=("Segoe UI", 12), width=30)
        course_name.grid(row=3, column=1, pady=6, padx=8)

        tk.Label(form_frame, text="Semester:", font=("Segoe UI", 12), bg="white").grid(row=4, column=0, sticky="e", pady=6, padx=8)
        semester = ttk.Combobox(form_frame, values=["1st Semester", "2nd Semester", "Summer"], state="readonly", font=("Segoe UI", 12), width=28)
        semester.grid(row=4, column=1, pady=6, padx=8)
        semester.current(0)

        tk.Label(form_frame, text="Units:", font=("Segoe UI", 12), bg="white").grid(row=5, column=0, sticky="e", pady=6, padx=8)
        units = ttk.Entry(form_frame, font=("Segoe UI", 12), width=30)
        units.grid(row=5, column=1, pady=6, padx=8)

        tk.Label(form_frame, text="Schedule:", font=("Segoe UI", 12), bg="white").grid(row=6, column=0, sticky="e", pady=6, padx=8)
        schedule = ttk.Entry(form_frame, font=("Segoe UI", 12), width=30)
        schedule.grid(row=6, column=1, pady=6, padx=8)

        tk.Label(form_frame, text="Instructor:", font=("Segoe UI", 12), bg="white").grid(row=7, column=0, sticky="e", pady=6, padx=8)
        instructor = ttk.Entry(form_frame, font=("Segoe UI", 12), width=30)
        instructor.grid(row=7, column=1, pady=6, padx=8)

        # --- Button Row ---
        button_frame = tk.Frame(form_frame, bg="white")
        button_frame.grid(row=8, column=0, columnspan=2, pady=20)

        def clear_fields():
            program.current(0)
            course_code.delete(0, tk.END)
            course_name.delete(0, tk.END)
            semester.current(0)
            units.delete(0, tk.END)
            schedule.delete(0, tk.END)
            instructor.delete(0, tk.END)

        def submit_enrollment():
            # Validate important fields
            if not all([
                program.get().strip(),
                course_code.get().strip(),
                course_name.get().strip(),
                semester.get().strip(),
                units.get().strip(),
                schedule.get().strip(),
                instructor.get().strip()
            ]):
                messagebox.showerror("Error", "All fields are required.")
                return

            # Save to in-memory storage (per session only)
            username = self.controller.current_user
            if not hasattr(self.controller, "enrollments"):
                self.controller.enrollments = {}
            if username not in self.controller.enrollments:
                self.controller.enrollments[username] = []
            self.controller.enrollments[username].append({
                "program": program.get().strip(),
                "course_code": course_code.get().strip(),
                "course_name": course_name.get().strip(),
                "semester": semester.get().strip(),
                "units": units.get().strip(),
                "schedule": schedule.get().strip(),
                "instructor": instructor.get().strip()
            })

            messagebox.showinfo("Success", "Enrollment submitted successfully!")
            clear_fields()

        tk.Button(
            button_frame,
            text="Clear Form",
            command=clear_fields,
            font=("Segoe UI", 11, "bold"),
            bg="#fbc02d",
            fg="white",
            activebackground="#f9a825",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack(side="left", padx=10)

        tk.Button(
            button_frame,
            text="Submit",
            command=submit_enrollment,
            font=("Segoe UI", 11, "bold"),
            bg="#388e3c",
            fg="white",
            activebackground="#2e7d32",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack(side="left", padx=10)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            font=("Segoe UI", 11, "bold"),
            bg="#d32f2f",
            fg="white",
            activebackground="#b71c1c",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack(side="left", padx=10)
        
    
    def show_schedule(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📅 Scheduled Items",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(pady=(10, 0), padx=30, fill="both", expand=True)

        # Define columns
        columns = ("Course", "Instructor")
        schedule_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=10)
        schedule_table.pack(fill="both", expand=True, padx=10, pady=10)

        for col in columns:
            schedule_table.heading(col, text=col)
            schedule_table.column(col, anchor="center", width=200)

        # Sample data
        scheduled_data = [
            ("Mathematics 101", "Prof. Alice Johnson"),
            ("Physics 201", "Dr. Mark Spencer"),
            ("History 301", "Dr. Emily Clark"),
            ("Computer Science 101", "Prof. John Smith"),
            ("Biology 102", "Prof. Susan Lee")
        ]

        for row in scheduled_data:
            schedule_table.insert("", "end", values=row)

        # Button frame below the table
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=20)

        def print_schedule():
            if not schedule_table.get_children():
                messagebox.showwarning("No Data", "No schedule data to print!")
            else:
                messagebox.showinfo("Print", "Schedule printed successfully!")

        tk.Button(
            button_frame,
            text="Print",
            command=print_schedule,
            font=("Segoe UI", 11, "bold"),
            bg="#388e3c",
            fg="white",
            activebackground="#2e7d32",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack(side="left", padx=15)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            font=("Segoe UI", 11, "bold"),
            bg="#d32f2f",
            fg="white",
            activebackground="#b71c1c",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack(side="left", padx=15)

    # ...existing code...
    def show_grades(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📈 Student Grades",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=20, pady=20)

        # Add Instructor column
        columns = ("Assignment", "Grade", "Instructor")
        grades_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=15)
        grades_table.pack(fill="both", expand=True)

        for col in columns:
            grades_table.heading(col, text=col)
            grades_table.column(col, width=200, anchor="center")

        # Get grades for the current student from shared grades dictionary
        username = self.controller.current_user
        student_grades = self.controller.grades.get(username, [])

        for entry in student_grades:
            # Show instructor if available, else "N/A"
            instructor = entry.get("instructor", "N/A")
            grades_table.insert("", "end", values=(entry["assignment"], entry["grade"], instructor))

        # Button frame
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=10)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            font=("Segoe UI", 11, "bold"),
            bg="#d32f2f",
            fg="white",
            activebackground="#b71c1c",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack(side="left", padx=15)
# ...existing code...

    
    def show_courses(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="📚 My Program Courses",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # Show info about the program (above table)
        username = self.controller.current_user
        student_data = self.controller.users["students"].get(username, {})
        program = None

        # Try to get the latest enrolled program from enrollments, else from registration
        if hasattr(self.controller, "enrollments"):
            user_enrollments = self.controller.enrollments.get(username, [])
            if user_enrollments:
                program = user_enrollments[-1].get("program")
        if not program:
            program = student_data.get("program", "Other")

        tk.Label(
            self.content_frame,
            text=f"Program: {program}",
            font=("Segoe UI", 12, "italic"),
            bg="white",
            fg="#555"
        ).pack(pady=(0, 10))

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=40, pady=(0, 20))

        columns = ("Course Code", "Course Name")
        courses_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=12)
        courses_table.pack(fill="both", expand=True)

        for col in columns:
            courses_table.heading(col, text=col)
            courses_table.column(col, width=300, anchor="center")

        # Sample data for each program
        courses_data = {
            "BS Computer Science": [
                ("CS101", "Introduction to Computing"),
                ("CS102", "Programming Fundamentals"),
                ("CS201", "Data Structures"),
                ("CS202", "Database Systems"),
                ("CS301", "Software Engineering")
            ],
            "BS Information Technology": [
                ("IT101", "IT Fundamentals"),
                ("IT102", "Web Development"),
                ("IT201", "Networking"),
                ("IT202", "System Administration"),
                ("IT301", "IT Project Management")
            ],
            "BS Engineering": [
                ("ENG101", "Engineering Math"),
                ("ENG102", "Thermodynamics"),
                ("ENG201", "Fluid Mechanics"),
                ("ENG202", "Materials Science"),
                ("ENG301", "Control Systems")
            ],
            "BS Business Administration": [
                ("BA101", "Principles of Management"),
                ("BA102", "Marketing Fundamentals"),
                ("BA201", "Financial Accounting"),
                ("BA202", "Business Law"),
                ("BA301", "Strategic Management")
            ],
            "BS Education": [
                ("EDU101", "Foundations of Education"),
                ("EDU102", "Child Development"),
                ("EDU201", "Curriculum Development"),
                ("EDU202", "Assessment and Evaluation"),
                ("EDU301", "Educational Technology")
            ],
            "BS Nursing": [
                ("NUR101", "Fundamentals of Nursing"),
                ("NUR102", "Anatomy and Physiology"),
                ("NUR201", "Medical-Surgical Nursing"),
                ("NUR202", "Community Health Nursing"),
                ("NUR301", "Nursing Leadership")
            ],
            "Other": [
                ("OTH101", "General Elective 1"),
                ("OTH102", "General Elective 2"),
                ("OTH201", "General Elective 3"),
                ("OTH202", "General Elective 4"),
                ("OTH301", "General Elective 5")
            ]
        }

        # Load courses for the student's program
        for course in courses_data.get(program, courses_data["Other"]):
            courses_table.insert("", "end", values=course)

        # Button frame below the table
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=10)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            font=("Segoe UI", 11, "bold"),
            bg="#d9534f",
            fg="white",
            activebackground="#b71c1c",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack()
        
    def show_instructors(self):
        self.clear_content()
        tk.Label(
            self.content_frame,
            text="👨‍🏫 Instructor Information",
            font=("Segoe UI", 16, "bold"),
            bg="white"
        ).pack(pady=10)

        # Table frame
        table_frame = tk.Frame(self.content_frame, bg="white")
        table_frame.pack(fill="both", expand=True, padx=30, pady=(0, 20))

        columns = ("Instructor Name", "Courses", "Description")
        instructor_table = ttk.Treeview(table_frame, columns=columns, show="headings", height=12)
        instructor_table.pack(fill="both", expand=True)

        for col in columns:
            instructor_table.heading(col, text=col)
            instructor_table.column(col, width=200 if col != "Description" else 350, anchor="center")

        # Sample data for instructors
        instructor_data = [
            ("Prof. Alice Johnson", "Mathematics 101", "Expert in calculus and algebra with 10+ years of teaching experience."),
            ("Dr. Mark Spencer", "Physics 201", "Specializes in theoretical physics and quantum mechanics."),
            ("Dr. Emily Clark", "History 301", "History scholar focusing on ancient civilizations and cultural studies."),
            ("Prof. John Smith", "Computer Science 101", "Passionate about AI and software development."),
            ("Prof. Susan Lee", "Biology 102", "Biology expert with research in genetics and molecular biology.")
        ]

        # Populate the table with data
        for row in instructor_data:
            instructor_table.insert("", "end", values=row)

        # Button frame below the table
        button_frame = tk.Frame(self.content_frame, bg="white")
        button_frame.pack(pady=10)

        tk.Button(
            button_frame,
            text="Exit",
            command=self.show_menu,
            font=("Segoe UI", 11, "bold"),
            bg="#d9534f",
            fg="white",
            activebackground="#b71c1c",
            activeforeground="white",
            relief="ridge",
            bd=0,
            padx=20,
            pady=8,
            cursor="hand2"
        ).pack()
        
    def back_to_login(self):
        if messagebox.askyesno("Logout", "Are you sure you want to logout?"):
            self.pack_forget()
            self.destroy()  # Ensure the frame is fully destroyed
            self.controller.frames.pop(StudentPortal, None)  # Remove from controller
            login_frame = self.controller.frames[LoginPage]
            login_frame.username_entry.delete(0, tk.END)
            login_frame.password_entry.delete(0, tk.END)
            login_frame.pack(fill="both", expand=True)
            self.controller.show_frame(LoginPage)


class StudentRegistrationForm(tk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent, bg="#f5f5f5")  # Light brown background for the whole form
        self.controller = controller
        self.users = controller.users

        # Title Section
        title = tk.Label(
            self,
            text="Student Registration Form",
            font=("Segoe UI", 20, "bold"),
            bg="#6a4e23",  # Brown color
            fg="white",
            pady=15
        )
        title.pack(fill="x")

        # Styling
        self.style = ttk.Style()
        self.style.theme_use('default')
        self.style.configure("TLabel", font=("Segoe UI", 10), background="#f5f5f5")
        self.style.configure("TButton", font=("Segoe UI", 10, "bold"), foreground="white", background="#8B4513")  # Darker brown
        self.style.map("TButton", background=[("active", "#6a4e23")])
        self.style.configure("TCombobox", padding=5)
        self.style.configure("TLabelframe.Label", font=("Segoe UI", 11, "bold"), foreground="#333")
        self.style.configure("TLabelframe", background="#f5f5f5", borderwidth=1, relief="solid")

        # Scrollable Canvas Setup
        canvas = tk.Canvas(self, bg="#f5f5f5", highlightthickness=0)
        scrollbar = tk.Scrollbar(self, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scrollbar.set)
        canvas.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        outer_frame = tk.Frame(canvas, bg="#f5f5f5")
        canvas.create_window((0, 0), window=outer_frame, anchor="n")
        outer_frame.bind("<Configure>", lambda e: canvas.configure(scrollregion=canvas.bbox("all")))
        canvas.bind_all("<MouseWheel>", lambda e: canvas.yview_scroll(-1 * (e.delta // 120), "units"))

        self.centered_frame = tk.Frame(outer_frame, bg="#f5f5f5")
        self.centered_frame.pack(anchor="center", pady=20)

        # Heading
        tk.Label(
            self.centered_frame,
            text="Student Registration Page",
            font=("Segoe UI", 24, "bold"),
            bg="#f5f5f5",
            fg="#4f4f4f"  # Darker gray text for a modern feel
        ).pack(pady=20)

        # Sections
        self.personal_info = self.create_section("Personal Information")
        self.contact_info = self.create_section("Contact Information")
        self.login_info = self.create_section("Login Credentials")

        # Populating the sections with input fields
        self.populate_personal_info()
        self.populate_contact_info()
        self.populate_login_info()

        # Button Container
        button_frame = tk.Frame(self.centered_frame, bg="#f5f5f5")
        button_frame.pack(pady=30)

        # Register Button
        ttk.Button(
            button_frame,
            text="REGISTER",
            command=self.register_student,
            style="TButton",
            width=20
        ).pack(pady=(0, 10))

        # Back to Login Button
        ttk.Button(
            button_frame,
            text="BACK TO LOGIN",
            command=self.back_to_login,
            style="TButton",
            width=20
        ).pack()

    def create_section(self, title):
        section = ttk.LabelFrame(self.centered_frame, text=title, padding=(20, 15))
        section.pack(padx=30, pady=15, fill="both", expand=True)
        return section

    def create_entry(self, parent, label_text):
        frame = tk.Frame(parent, bg="#f5f5f5")
        frame.pack(fill="x", pady=5)
        ttk.Label(frame, text=label_text, width=25).pack(side="left")
        entry = ttk.Entry(frame)
        entry.pack(side="right", expand=True, fill="x")
        return entry

    def populate_personal_info(self):
        self.first_name = self.create_entry(self.personal_info, "First Name")
        self.last_name = self.create_entry(self.personal_info, "Last Name")
        self.dob = self.create_entry(self.personal_info, "Date of Birth")

    def populate_contact_info(self):
        self.email = self.create_entry(self.contact_info, "Email Address")
        self.phone = self.create_entry(self.contact_info, "Phone Number")

    def populate_login_info(self):
        self.username = self.create_entry(self.login_info, "Username")
        self.password = self.create_entry(self.login_info, "Password")
        self.password.config(show="*")
        
    def register_student(self):
        username = self.username.get().strip()
        password = self.password.get().strip()
        email = self.email.get().strip()
        phone = self.phone.get().strip()
        first_name = self.first_name.get().strip()
        last_name = self.last_name.get().strip()

        if not username or not password or not email or not first_name or not last_name:
            messagebox.showerror("Error", "All fields are required.")
            return

        if username in self.users["students"] or username in self.users["teachers"]:
            messagebox.showerror("Error", "Username already exists.")
            return

        # Add student details to the users dictionary
        self.users["students"][username] = {
            "password": password,
            "email": email,
            "phone": phone,
            "first_name": first_name,
            "last_name": last_name,
            "course": "Computer Science",  # Example course
            "year_level": "3rd Year"       # Example year level
        }

        messagebox.showinfo("Success", f"Student '{username}' registered successfully!")
        self.back_to_login()
        
    def back_to_login(self):
        self.pack_forget()
        self.destroy()  # <-- ADD THIS to remove the registration form
        # REMOVE the registration frame from frames dictionary
        self.controller.frames.pop(StudentRegistrationForm, None)

        login_frame = self.controller.frames[LoginPage]
        login_frame.pack(fill="both", expand=True)
        self.controller.show_frame(LoginPage)


class TeacherRegistrationForm(tk.Frame):
    def __init__(self, parent, controller):
        super().__init__(parent)
        self.controller = controller
        self.users = controller.users

        title = tk.Label(
            self,
            text="Teacher Registration Module",
            font=("Segoe UI", 20, "bold"),
            bg="#8B4513",  # Brown header
            fg="white",
            pady=15
        )
        title.pack(fill="x")

        self.style = ttk.Style()
        self.style.theme_use('default')
        self.style.configure("TLabel", font=("Segoe UI", 10))
        self.style.configure("TButton", font=("Segoe UI", 10, "bold"), foreground="white", background="#A0522D")
        self.style.map("TButton", background=[("active", "#7B3F00")])
        self.style.configure("TCombobox", padding=5)
        self.style.configure("TLabelframe.Label", font=("Segoe UI", 11, "bold"))
        self.style.configure("TLabelframe", borderwidth=1, relief="solid")

        canvas = tk.Canvas(self, highlightthickness=0)
        scrollbar = tk.Scrollbar(self, orient="vertical", command=canvas.yview)
        canvas.configure(yscrollcommand=scrollbar.set)
        canvas.pack(side="left", fill="both", expand=True)
        scrollbar.pack(side="right", fill="y")

        outer_frame = tk.Frame(canvas)
        canvas.create_window((0, 0), window=outer_frame, anchor="n")
        outer_frame.bind("<Configure>", lambda e: canvas.configure(scrollregion=canvas.bbox("all")))
        canvas.bind_all("<MouseWheel>", lambda e: canvas.yview_scroll(-1 * (e.delta // 120), "units"))

        self.centered_frame = tk.Frame(outer_frame)
        self.centered_frame.pack(anchor="center", pady=20)

        tk.Label(
            self.centered_frame,
            text="Teacher Registration Portal",
            font=("Segoe UI", 24, "bold")
        ).pack(pady=20)

        self.personal_info = self.create_section("Personal Information")
        self.professional_info = self.create_section("Professional Information")
        self.contact_info = self.create_section("Contact Information")
        self.login_info = self.create_section("Login Credentials")

        self.populate_personal_info()
        self.populate_professional_info()
        self.populate_contact_info()
        self.populate_login_info()

        # Right-aligned buttons
        button_frame = tk.Frame(self.centered_frame)
        button_frame.pack(fill="x", padx=30, pady=(20, 10), anchor="e")

        ttk.Button(
            button_frame,
            text="REGISTER",
            command=self.register_teacher,
            style="TButton",
            width=18
        ).pack(side="right", padx=10)

        ttk.Button(
            button_frame,
            text="BACK TO LOGIN",
            command=self.back_to_login,
            style="TButton",
            width=18
        ).pack(side="right")

    def create_section(self, title):
        section = ttk.LabelFrame(self.centered_frame, text=title, padding=(20, 15))
        section.pack(padx=30, pady=20, fill="both", expand=True)
        return section

    def create_entry(self, parent, label_text):
        frame = tk.Frame(parent)
        frame.pack(fill="x", pady=5)
        ttk.Label(frame, text=label_text, width=25).pack(side="left")
        entry = ttk.Entry(frame)
        entry.pack(side="right", expand=True, fill="x", padx=5)
        return entry

    def create_combobox(self, parent, label_text, values):
        frame = tk.Frame(parent)
        frame.pack(fill="x", pady=5)
        ttk.Label(frame, text=label_text, width=25).pack(side="left")
        combo = ttk.Combobox(frame, values=values, state="readonly")
        combo.current(0)
        combo.pack(side="right", expand=True, fill="x", padx=5)
        return combo

    def populate_personal_info(self):
        self.first_name = self.create_entry(self.personal_info, "First Name")
        self.last_name = self.create_entry(self.personal_info, "Last Name")
        self.gender = self.create_combobox(self.personal_info, "Gender", ["Male", "Female", "Other"])
        self.dob = self.create_entry(self.personal_info, "Date of Birth")
        self.nationality = self.create_entry(self.personal_info, "Nationality")

    def populate_professional_info(self):
        self.employee_id = self.create_entry(self.professional_info, "Employee ID")
        self.department = self.create_entry(self.professional_info, "Department")
        self.subjects = self.create_entry(self.professional_info, "Subjects Taught")
        self.years_of_experience = self.create_entry(self.professional_info, "Years of Experience")
        self.highest_qualification = self.create_entry(self.professional_info, "Highest Qualification")

    def populate_contact_info(self):
        self.email = self.create_entry(self.contact_info, "Email Address")
        self.phone = self.create_entry(self.contact_info, "Phone Number")
        self.address = self.create_entry(self.contact_info, "Address")

    def populate_login_info(self):
        self.username = self.create_entry(self.login_info, "Username")
        self.password = self.create_entry(self.login_info, "Password")
        self.password.config(show="*")

    def register_teacher(self):
        username = self.username.get().strip()
        password = self.password.get().strip()

        if not username or not password:
            messagebox.showerror("Error", "Username and Password are required.")
            return

        if username in self.users["teachers"] or username in self.users["students"]:
            messagebox.showerror("Error", "Username already exists.")
            return

        self.users["teachers"][username] = {
            "password": password,
            "email": self.email.get().strip(),
            "phone": self.phone.get().strip(),
            "first_name": self.first_name.get().strip(),
            "last_name": self.last_name.get().strip(),
            "department": self.department.get().strip(),
            "subjects": self.subjects.get().strip(),
            "years_of_experience": self.years_of_experience.get().strip()
        }

        messagebox.showinfo("Success", f"Teacher '{username}' registered successfully!")
        self.back_to_login()

    def back_to_login(self):
        self.pack_forget()
        self.destroy()
        self.controller.frames.pop(TeacherRegistrationForm, None)

        login_frame = self.controller.frames[LoginPage]
        login_frame.pack(fill="both", expand=True)
        self.controller.show_frame(LoginPage)
        
if __name__ == "__main__":
    root = tk.Tk()
    root.title("KLL Management System")
    root.geometry("1024x768")
    root.configure(bg="#ffffff")

    container = tk.Frame(root)
    container.pack(fill="both", expand=True)

    app = AppController(root, container)
    root.mainloop()
