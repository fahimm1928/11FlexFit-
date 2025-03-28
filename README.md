import SwiftUI
import Foundation
import Charts

// mdoels 

// User model 
class User: ObservableObject {
    @Published var name: String
    @Published var age: Int
    @Published var weight: Double
    @Published var weightGoal: String
    @Published var fitnessLevel: String
    @Published var points: Int = 0
    @Published var weightHistory: [(Date, Double)] = []
    @Published var gymDays: [Date] = []
    @Published var exerciseLog: [Date: [ExerciseEntry]] = [:]
    
    init(name: String, age: Int, weight: Double, weightGoal: String, fitnessLevel: String) {
        self.name = name
        self.age = age
        self.weight = weight
        self.weightGoal = weightGoal
        self.fitnessLevel = fitnessLevel
        self.weightHistory = [(Date(), weight)]
    }
}

// Exercise entry logging
struct ExerciseEntry: Identifiable {
    let id = UUID()
    let name: String
    let sets: Int
    let reps: Int
}

// Leaderboard to manage leadership
class Leaderboard: ObservableObject {
    @Published var entries: [(name: String, points: Int)]
    
    init() {
        entries = [
            ("Mohammed Louchini", Int.random(in: 100...500)),
            ("Mohammed Sakhi", Int.random(in: 100...500)),
            ("Anas Omar", Int.random(in: 100...500))
        ]
    }
    
    func updateUserPoints(user: User) {
        if let index = entries.firstIndex(where: { $0.name == user.name }) {
            entries[index].points = user.points
        } else {
            entries.append((user.name, user.points))
        }
        entries.sort { $0.points > $1.points } // Sort descending by points
    }
}

// Challenge for mini-games
struct Challenge: Identifiable {
    let id = UUID()
    let description: String
    let points: Int
    let duration: TimeInterval? // Optional duration in seconds for timed challenges
}

// Predefined data
let challenges = [
    Challenge(description: "Complete a 1-minute plank", points: 20, duration: 60),
    Challenge(description: "Do 20 push-ups", points: 15, duration: nil),
    Challenge(description: "Run for 10 minutes", points: 25, duration: nil),
    Challenge(description: "Hold a wall sit for 30 seconds", points: 10, duration: 30),
    Challenge(description: "Complete 50 jumping jacks", points: 20, duration: nil)
]

let exercises = ["Push-up", "Squat", "Plank", "Bench Press", "Deadlift", "Pull-up"]

// MARK: - Views

// Sign-up view with validation
struct SignUpView: View {
    @State private var name = ""
    @State private var age = ""
    @State private var weight = ""
    @State private var weightGoal = "Lose Weight"
    @State private var fitnessLevel = "Beginner"
    @State private var errorMessage: String?
    
    let weightGoals = ["Lose Weight", "Maintain Weight", "Gain Muscle"]
    let fitnessLevels = ["Beginner", "Intermediate", "Advanced"]
    
    var onSignUp: (User) -> Void
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Sign Up").font(.title).bold()
            TextField("Name", text: $name)
                .textFieldStyle(RoundedBorderTextFieldStyle())
            TextField("Age", text: $age)
                .keyboardType(.numberPad)
                .textFieldStyle(RoundedBorderTextFieldStyle())
            TextField("Weight (kg)", text: $weight)
                .keyboardType(.decimalPad)
                .textFieldStyle(RoundedBorderTextFieldStyle())
            Picker("Weight Goal", selection: $weightGoal) {
                ForEach(weightGoals, id: \.self) { goal in
                    Text(goal).tag(goal)
                }
            }.pickerStyle(WheelPickerStyle())
            Picker("Fitness Level", selection: $fitnessLevel) {
                ForEach(fitnessLevels, id: \.self) { level in
                    Text(level).tag(level)
                }
            }.pickerStyle(WheelPickerStyle())
            Button("Sign Up") {
                if validateInputs() {
                    let user = User(name: name, age: Int(age) ?? 0, weight: Double(weight) ?? 0, weightGoal: weightGoal, fitnessLevel: fitnessLevel)
                    onSignUp(user)
                }
            }
            .buttonStyle(.borderedProminent)
            if let error = errorMessage {
                Text(error).foregroundColor(.red)
            }
        }
        .padding()
    }
    
    func validateInputs() -> Bool {
        guard !name.isEmpty else {
            errorMessage = "Name is required"
            return false
        }
        guard let ageInt = Int(age), ageInt > 15 else {
            errorMessage = "Age must be above 15"
            return false
        }
        guard let weightDouble = Double(weight), weightDouble > 0 else {
            errorMessage = "Invalid weight"
            return false
        }
        errorMessage = nil
        return true
    }
}

// Main view with tabs
struct MainView: View {
    @ObservedObject var user: User
    @StateObject var leaderboard = Leaderboard()
    
    var body: some View {
        TabView {
            LogView(user: user)
                .tabItem { Label("Log", systemImage: "list.bullet") }
            CalendarView(user: user)
                .tabItem { Label("Calendar", systemImage: "calendar") }
            LeaderboardView(leaderboard: leaderboard, user: user)
                .tabItem { Label("Leaderboard", systemImage: "trophy") }
            ChallengesView(user: user)
                .tabItem { Label("Challenges", systemImage: "gamecontroller") }
            ProfileView(user: user)
                .tabItem { Label("Profile", systemImage: "person") }
        }
        .onAppear {
            leaderboard.updateUserPoints(user: user)
        }
    }
}

// Log view for exercise input
struct LogView: View {
    @ObservedObject var user: User
    @State private var selectedDate = Date()
    @State private var searchText = ""
    @State private var selectedExercise: String?
    @State private var sets = 1
    @State private var reps = 10
    
    var filteredExercises: [String] {
        searchText.isEmpty ? exercises : exercises.filter { $0.lowercased().contains(searchText.lowercased()) }
    }
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Exercise Log").font(.title2).bold()
            DatePicker("Select Date", selection: $selectedDate, displayedComponents: .date)
            TextField("Search Exercise", text: $searchText)
                .textFieldStyle(RoundedBorderTextFieldStyle())
            List(filteredExercises, id: \.self) { exercise in
                Button(exercise) { selectedExercise = exercise }
            }
            if let exercise = selectedExercise {
                Text("Selected Exercise: \(exercise)")
                    .font(.headline)
                VStack(alignment: .leading, spacing: 10) {
                    HStack {
                        Text("Sets:")
                        Picker("", selection: $sets) {
                            ForEach(1...20, id: \.self) { num in
                                Text("\(num)").tag(num)
                            }
                        }.pickerStyle(WheelPickerStyle())
                            .frame(width: 100, height: 100)
                    }
                    HStack {
                        Text("Reps:")
                        Picker("", selection: $reps) {
                            ForEach(1...20, id: \.self) { num in
                                Text("\(num)").tag(num)
                            }
                        }.pickerStyle(WheelPickerStyle())
                            .frame(width: 100, height: 100)
                    }
                }
                Button("Add to Log") {
                    let entry = ExerciseEntry(name: exercise, sets: sets, reps: reps)
                    if user.exerciseLog[selectedDate] == nil {
                        user.exerciseLog[selectedDate] = []
                    }
                    user.exerciseLog[selectedDate]?.append(entry)
                    selectedExercise = nil
                }
                .buttonStyle(.borderedProminent)
            }
            if let logs = user.exerciseLog[selectedDate] {
                List(logs) { log in
                    Text("\(log.name): \(log.sets) sets of \(log.reps) reps")
                }
            }
        }
        .padding()
    }
}

// Calendar view for marking gym days
struct CalendarView: View {
    @ObservedObject var user: User
    @State private var selectedDate = Date()
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Gym Calendar").font(.title2).bold()
            DatePicker("Select Gym Day", selection: $selectedDate, displayedComponents: .date)
            Button("Mark as Gym Day") {
                if !user.gymDays.contains(selectedDate) {
                    user.gymDays.append(selectedDate)
                }
            }
            .buttonStyle(.borderedProminent)
            List(user.gymDays.sorted(), id: \.self) { day in
                Text(day, style: .date)
            }
        }
        .padding()
    }
}

// Leaderboard view
struct LeaderboardView: View {
    @ObservedObject var leaderboard: Leaderboard
    @ObservedObject var user: User
    
    var body: some View {
        VStack {
            Text("Leaderboard").font(.title2).bold()
            List(leaderboard.entries, id: \.name) { entry in
                Text("\(entry.name): \(entry.points) points")
            }
        }
        .onAppear {
            leaderboard.updateUserPoints(user: user)
        }
        .padding()
    }
}

// Challenges view for mini-games
struct ChallengesView: View {
    @ObservedObject var user: User
    @State private var selectedChallenge: Challenge?
    @State private var timeRemaining: TimeInterval = 0
    @State private var isTimerRunning = false
    @State private var timer: Timer?
    @State private var showAlert = false
    @State private var alertMessage = ""
    
    var body: some View {
        VStack {
            Text("Challenges").font(.title2).bold()
            List(challenges) { challenge in
                Button(challenge.description) {
                    selectedChallenge = challenge
                }
            }
        }
        .sheet(item: $selectedChallenge) { challenge in
            VStack(spacing: 20) {
                Text(challenge.description)
                    .font(.headline)
                if let duration = challenge.duration {
                    Text("Time Remaining: \(Int(timeRemaining)) seconds")
                    if !isTimerRunning {
                        Button("Start Timer") {
                            timeRemaining = duration
                            isTimerRunning = true
                            timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
                                if timeRemaining > 0 {
                                    timeRemaining -= 1
                                } else {
                                    timer?.invalidate()
                                    isTimerRunning = false
                                    user.points += challenge.points
                                    alertMessage = "Challenge Completed! +\(challenge.points) points"
                                    showAlert = true
                                    selectedChallenge = nil
                                }
                            }
                        }
                        .buttonStyle(.borderedProminent)
                    } else {
                        Button("Cancel") {
                            timer?.invalidate()
                            isTimerRunning = false
                            alertMessage = "Challenge Failed"
                            showAlert = true
                            selectedChallenge = nil
                        }
                        .buttonStyle(.bordered)
                        .foregroundColor(.red)
                    }
                } else {
                    Button("Complete Challenge") {
                        user.points += challenge.points
                        alertMessage = "Challenge Completed! +\(challenge.points) points"
                        showAlert = true
                        selectedChallenge = nil
                    }
                    .buttonStyle(.borderedProminent)
                }
            }
            .alert(isPresented: $showAlert) {
                Alert(title: Text(alertMessage), dismissButton: .default(Text("OK")))
            }
            .padding()
        }
        .padding()
    }
}

// Profile view with weight and age tracking, including graph
struct ProfileView: View {
    @ObservedObject var user: User
    @State private var newWeight = ""
    @State private var newAge = ""
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Profile").font(.title2).bold()
            Text("Name: \(user.name)")
            Text("Age: \(user.age)")
            Text("Weight: \(user.weight) kg")
            Text("Weight Goal: \(user.weightGoal)")
            Text("Fitness Level: \(user.fitnessLevel)")
            Text("Points: \(user.points)")
            TextField("Update Weight (kg)", text: $newWeight)
                .keyboardType(.decimalPad)
                .textFieldStyle(RoundedBorderTextFieldStyle())
            TextField("Update Age", text: $newAge)
                .keyboardType(.numberPad)
                .textFieldStyle(RoundedBorderTextFieldStyle())
            Button("Save Updates") {
                if let weight = Double(newWeight) {
                    user.weight = weight
                    user.weightHistory.append((Date(), weight))
                    newWeight = ""
                }
                if let age = Int(newAge) {
                    user.age = age
                    newAge = ""
                }
            }
            .buttonStyle(.borderedProminent)
            Text("Weight History").font(.headline)
            Chart {
                ForEach(user.weightHistory.sorted(by: { $0.0 < $1.0 }), id: \.0) { entry in
                    LineMark(
                        x: .value("Date", entry.0),
                        y: .value("Weight (kg)", entry.1)
                    )
                    .foregroundStyle(.blue)
                    PointMark(
                        x: .value("Date", entry.0),
                        y: .value("Weight (kg)", entry.1)
                    )
                    .foregroundStyle(.blue)
                }
            }
            .frame(height: 200)
            .padding()
        }
        .padding()
    }
}

// MARK: - App Entry Point

// Root content view
struct ContentView: View {
    @State private var user: User?
    @State private var leaderboard: Leaderboard?
    
    var body: some View {
        if let user = user, let leaderboard = leaderboard {
            MainView(user: user, leaderboard: leaderboard)
        } else {
            SignUpView { newUser in
                user = newUser
                leaderboard = Leaderboard()
                leaderboard?.updateUserPoints(user: newUser)
            }
        }
    }
}

// App entry point
struct FitnessApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
