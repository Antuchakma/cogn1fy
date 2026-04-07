# Antu Contribution Guide

This contribution keeps the shared Xcode project and asset catalog unchanged and splits the Swift source files by feature ownership.

## Feature Split
- App shell and tab navigation
- Home, upload, and document detail UX
- Flashcard study, quiz taking, and supporting views

## Folder Creation
Create folder inside cognify folder: `cognify`.

## File Creation
### cognify/cognifyApp.swift
Create file `cognifyApp.swift` inside `cognify` folder.
```swift
//
//  cognifyApp.swift
//  cognify
//
//  Created by Kusalab Dewan on 23/3/2026.
//

import SwiftUI

@main
struct cognifyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### cognify/ContentView.swift
Create file `ContentView.swift` inside `cognify` folder.
```swift
//
//  ContentView.swift
//  cognify
//
//  Created by Kusalab Dewan on 23/3/2026.
//

import SwiftUI
import PhotosUI

struct ContentView: View {
    @State private var selectedTab = 0
    @State private var dataManager = DataManager.shared
    
    var body: some View {
        TabView(selection: $selectedTab) {
            HomeView()
                .tabItem {
                    Label("Home", systemImage: "house.fill")
                }
                .tag(0)
            
            UploadView()
                .tabItem {
                    Label("Upload", systemImage: "plus.circle.fill")
                }
                .tag(1)
            
            FlashcardsView()
                .tabItem {
                    Label("Flashcards", systemImage: "rectangle.stack.fill")
                }
                .tag(2)
            
            QuizView()
                .tabItem {
                    Label("Quiz", systemImage: "questionmark.circle.fill")
                }
                .tag(3)
            
            ProgressDashboardView()
                .tabItem {
                    Label("Progress", systemImage: "chart.bar.fill")
                }
                .tag(4)
        }
        .environment(dataManager)
    }
}

// MARK: - Home View
struct HomeView: View {
    @Environment(DataManager.self) private var dataManager
    @State private var selectedDocument: Document?
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 20) {
                    // Welcome Header
                    HStack {
                        VStack(alignment: .leading, spacing: 4) {
                            Text("Welcome back!")
                                .font(.title2)
                                .fontWeight(.semibold)
                            Text("Ready to study smarter?")
                                .font(.subheadline)
                                .foregroundStyle(.secondary)
                        }
                        Spacer()
                        Image(systemName: "brain.head.profile")
                            .font(.system(size: 40))
                            .foregroundStyle(.tint)
                    }
                    .padding()
                    
                    // Quick Stats
                    HStack(spacing: 12) {
                        QuickStatCard(value: "\(dataManager.documents.count)", label: "Documents", color: .blue)
                        QuickStatCard(value: "\(dataManager.flashcards.count)", label: "Flashcards", color: .purple)
                        QuickStatCard(value: "\(dataManager.progress.currentStreak)", label: "Day Streak", color: .orange)
                    }
                    .padding(.horizontal)
                    
                    // Recent Documents
                    VStack(alignment: .leading, spacing: 12) {
                        Text("Recent Documents")
                            .font(.headline)
                            .padding(.horizontal)
                        
                        if dataManager.documents.isEmpty {
                            VStack(spacing: 8) {
                                Text("No documents yet")
                                    .foregroundStyle(.secondary)
                                Text("Upload your first study material to get started!")
                                    .font(.caption)
                                    .foregroundStyle(.tertiary)
                            }
                            .frame(maxWidth: .infinity)
                            .padding()
                            .background(.quaternary.opacity(0.5))
                            .clipShape(RoundedRectangle(cornerRadius: 12))
                            .padding(.horizontal)
                        } else {
                            ForEach(dataManager.documents.prefix(5)) { document in
                                DocumentRow(document: document)
                                    .onTapGesture {
                                        selectedDocument = document
                                    }
                                    .padding(.horizontal)
                            }
                        }
                    }
                }
                .padding(.vertical)
            }
            .navigationTitle("cognify")
            .navigationDestination(item: $selectedDocument) { document in
                DocumentDetailView(document: document)
            }
        }
    }
}

struct QuickStatCard: View {
    let value: String
    let label: String
    let color: Color
    
    var body: some View {
        VStack(spacing: 4) {
            Text(value)
                .font(.title2)
                .fontWeight(.bold)
                .foregroundStyle(color)
            Text(label)
                .font(.caption2)
                .foregroundStyle(.secondary)
        }
        .frame(maxWidth: .infinity)
        .padding(.vertical, 12)
        .background(color.opacity(0.1))
        .clipShape(RoundedRectangle(cornerRadius: 10))
    }
}

struct DocumentRow: View {
    let document: Document
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: iconForDocumentType(document.type))
                .font(.title2)
                .foregroundStyle(.blue)
                .frame(width: 40, height: 40)
                .background(.blue.opacity(0.1))
                .clipShape(RoundedRectangle(cornerRadius: 8))
            
            VStack(alignment: .leading, spacing: 4) {
                Text(document.title)
                    .font(.subheadline)
                    .fontWeight(.medium)
                Text(document.createdAt, style: .date)
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }
            
            Spacer()
            
            if document.processedByAI {
                Image(systemName: "checkmark.circle.fill")
                    .foregroundStyle(.green)
            }
            
            Image(systemName: "chevron.right")
                .font(.caption)
                .foregroundStyle(.tertiary)
        }
        .padding()
        .background(Color(.systemBackground))
        .clipShape(RoundedRectangle(cornerRadius: 12))
        .shadow(color: .black.opacity(0.05), radius: 5, y: 2)
    }
    
    private func iconForDocumentType(_ type: DocumentType) -> String {
        switch type {
        case .pdf: return "doc.fill"
        case .image: return "photo.fill"
        case .text: return "doc.text.fill"
        }
    }
}

// MARK: - Upload View
struct UploadView: View {
    @Environment(DataManager.self) private var dataManager
    @State private var selectedImages: [PhotosPickerItem] = []
    @State private var showingCamera = false
    @State private var showingTextInput = false
    @State private var showingPDFPicker = false
    @State private var isProcessing = false
    @State private var processingStatus = ""
    @State private var showingError = false
    @State private var errorMessage = ""
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                if isProcessing {
                    ProgressView(processingStatus)
                        .padding()
                } else {
                    Spacer()
                    
                    Image(systemName: "arrow.up.doc.fill")
                        .font(.system(size: 60))
                        .foregroundStyle(.tint)
                    
                    Text("Upload Study Material")
                        .font(.title2)
                        .fontWeight(.semibold)
                    
                    Text("PDFs, images, or paste text")
                        .font(.subheadline)
                        .foregroundStyle(.secondary)
                    
                    VStack(spacing: 12) {
                        Button(action: { showingPDFPicker = true }) {
                            Label("Choose PDF", systemImage: "doc.fill")
                                .frame(maxWidth: .infinity)
                        }
                        .buttonStyle(.borderedProminent)
                        
                        Button(action: { showingCamera = true }) {
                            Label("Take Photo", systemImage: "camera.fill")
                                .frame(maxWidth: .infinity)
                        }
                        .buttonStyle(.borderedProminent)
                        
                        PhotosPicker(selection: $selectedImages, matching: .images) {
                            Label("Choose Image", systemImage: "photo.fill")
                                .frame(maxWidth: .infinity)
                        }
                        .buttonStyle(.borderedProminent)
                        
                        Button(action: { showingTextInput = true }) {
                            Label("Paste Text", systemImage: "doc.text.fill")
                                .frame(maxWidth: .infinity)
                        }
                        .buttonStyle(.bordered)
                    }
                    .padding(.horizontal, 40)
                    
                    Spacer()
                }
            }
            .navigationTitle("Upload")
            .sheet(isPresented: $showingCamera) {
                CameraView { image in
                    processImage(image)
                }
            }
            .sheet(isPresented: $showingTextInput) {
                TextInputView { text in
                    processText(text)
                }
            }
            .fileImporter(isPresented: $showingPDFPicker, allowedContentTypes: [.pdf]) { result in
                handlePDFSelection(result)
            }
            .onChange(of: selectedImages) { oldValue, newValue in
                if !newValue.isEmpty {
                    Task {
                        await processSelectedImages()
                    }
                }
            }
            .alert("Error", isPresented: $showingError) {
                Button("OK", role: .cancel) {}
            } message: {
                Text(errorMessage)
            }
        }
    }
    
    private func processImage(_ image: UIImage) {
        Task {
            isProcessing = true
            processingStatus = "Extracting text from image..."
            
            do {
                let extractedText = try await OCRService.shared.extractText(from: image)
                
                // Create document
                let imageData = image.jpegData(compressionQuality: 0.7)
                let document = Document(
                    title: "Scanned Image \(Date().formatted(date: .abbreviated, time: .shortened))",
                    content: extractedText,
                    type: .image,
                    thumbnailData: imageData
                )
                
                dataManager.addDocument(document)
                
                // Process with AI
                await processWithAI(document)
                
                isProcessing = false
                processingStatus = ""
            } catch {
                isProcessing = false
                showError(error.localizedDescription)
            }
        }
    }
    
    private func processSelectedImages() async {
        isProcessing = true
        processingStatus = "Processing images..."
        
        var images: [UIImage] = []
        
        for item in selectedImages {
            if let data = try? await item.loadTransferable(type: Data.self),
               let image = UIImage(data: data) {
                images.append(image)
            }
        }
        
        selectedImages.removeAll()
        
        guard !images.isEmpty else {
            isProcessing = false
            showError("Failed to load images")
            return
        }
        
        do {
            processingStatus = "Extracting text..."
            let extractedText = try await OCRService.shared.extractText(from: images)
            
            let thumbnailData = images.first?.jpegData(compressionQuality: 0.5)
            let document = Document(
                title: "Images \(Date().formatted(date: .abbreviated, time: .shortened))",
                content: extractedText,
                type: .image,
                thumbnailData: thumbnailData
            )
            
            dataManager.addDocument(document)
            
            await processWithAI(document)
            
            isProcessing = false
            processingStatus = ""
        } catch {
            isProcessing = false
            showError(error.localizedDescription)
        }
    }
    
    private func processText(_ text: String) {
        guard !text.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else {
            showError("Please enter some text")
            return
        }
        
        Task {
            isProcessing = true
            processingStatus = "Processing text..."
            
            let document = Document(
                title: "Text Note \(Date().formatted(date: .abbreviated, time: .shortened))",
                content: text,
                type: .text
            )
            
            dataManager.addDocument(document)
            
            await processWithAI(document)
            
            isProcessing = false
            processingStatus = ""
        }
    }
    
    private func handlePDFSelection(_ result: Result<URL, Error>) {
        switch result {
        case .success(let url):
            processPDF(url)
        case .failure(let error):
            showError(error.localizedDescription)
        }
    }
    
    private func processPDF(_ url: URL) {
        Task {
            isProcessing = true
            processingStatus = "Processing PDF..."
            
            // In production, you'd extract text from PDF using PDFKit
            // For now, placeholder
            let document = Document(
                title: url.lastPathComponent,
                content: "PDF content would be extracted here",
                type: .pdf
            )
            
            dataManager.addDocument(document)
            
            await processWithAI(document)
            
            isProcessing = false
            processingStatus = ""
        }
    }
    
    private func processWithAI(_ document: Document) async {
        processingStatus = "Generating flashcards and quiz..."
        
        do {
            // Generate flashcards - use fallback for demo mode
            let flashcardResponses: [FlashcardResponse]
            if NetworkService.USE_MOCK_DATA {
                flashcardResponses = try await MockDataService.shared.generateFlashcards(from: document.content, count: 10)
            } else {
                flashcardResponses = try await NetworkService.shared.generateFlashcards(from: document.content, count: 10)
            }
            
            let flashcards = flashcardResponses.map { response in
                Flashcard(
                    question: response.question,
                    answer: response.answer,
                    documentId: document.id,
                    difficulty: parseDifficulty(response.difficulty)
                )
            }
            dataManager.addFlashcards(flashcards)
            
            // Generate quiz - use fallback for demo mode
            let quizResponses: [QuizQuestionResponse]
            if NetworkService.USE_MOCK_DATA {
                quizResponses = try await MockDataService.shared.generateQuiz(from: document.content, questionCount: 5)
            } else {
                quizResponses = try await NetworkService.shared.generateQuiz(from: document.content, questionCount: 5)
            }
            
            let quizQuestions = quizResponses.map { response in
                QuizQuestion(
                    question: response.question,
                    options: response.options,
                    correctAnswer: response.correctAnswer,
                    explanation: response.explanation
                )
            }
            
            let quiz = Quiz(
                title: "Quiz: \(document.title)",
                documentId: document.id,
                questions: quizQuestions
            )
            dataManager.addQuiz(quiz)
            
            // Mark document as processed
            if let index = dataManager.documents.firstIndex(where: { $0.id == document.id }) {
                dataManager.documents[index].processedByAI = true
            }
            
        } catch {
            // AI processing failed, but document is still saved
            print("AI processing failed: \(error.localizedDescription)")
            // Could show a non-blocking notification here
        }
    }
    
    private func parseDifficulty(_ difficultyString: String?) -> Difficulty {
        guard let difficultyString = difficultyString?.lowercased() else { return .medium }
        switch difficultyString {
        case "easy": return .easy
        case "hard": return .hard
        default: return .medium
        }
    }
    
    private func showError(_ message: String) {
        errorMessage = message
        showingError = true
    }
}

// MARK: - Flashcards View
struct FlashcardsView: View {
    @Environment(DataManager.self) private var dataManager
    @State private var selectedDocument: Document?
    @State private var showingStudyMode = false
    
    var body: some View {
        NavigationStack {
            Group {
                if dataManager.flashcards.isEmpty {
                    ContentUnavailableView(
                        "No flashcards yet",
                        systemImage: "rectangle.stack",
                        description: Text("Upload materials to generate flashcards")
                    )
                } else {
                    ScrollView {
                        VStack(spacing: 16) {
                            // Study Due Section
                            let dueCards = dataManager.getFlashcardsDueForReview()
                            if !dueCards.isEmpty {
                                VStack(alignment: .leading, spacing: 12) {
                                    HStack {
                                        Text("Due for Review")
                                            .font(.headline)
                                        Spacer()
                                        Text("\(dueCards.count) cards")
                                            .font(.caption)
                                            .foregroundStyle(.secondary)
                                    }
                                    
                                    Button(action: { showingStudyMode = true }) {
                                        HStack {
                                            Image(systemName: "brain.head.profile")
                                            Text("Start Review Session")
                                            Spacer()
                                            Image(systemName: "arrow.right")
                                        }
                                        .padding()
                                        .background(.blue.gradient)
                                        .foregroundStyle(.white)
                                        .clipShape(RoundedRectangle(cornerRadius: 12))
                                    }
                                }
                                .padding()
                            }
                            
                            // Flashcards by Document
                            ForEach(dataManager.documents) { document in
                                let flashcards = dataManager.getFlashcards(for: document.id)
                                if !flashcards.isEmpty {
                                    DocumentFlashcardSection(document: document, flashcards: flashcards)
                                        .padding(.horizontal)
                                }
                            }
                        }
                        .padding(.vertical)
                    }
                }
            }
            .navigationTitle("Flashcards")
            .sheet(isPresented: $showingStudyMode) {
                FlashcardStudyView(flashcards: dataManager.getFlashcardsDueForReview())
            }
        }
    }
}

struct DocumentFlashcardSection: View {
    let document: Document
    let flashcards: [Flashcard]
    @State private var showingAllCards = false
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                Text(document.title)
                    .font(.headline)
                Spacer()
                Text("\(flashcards.count) cards")
                    .font(.caption)
                    .foregroundStyle(.secondary)
            }
            
            Button(action: { showingAllCards = true }) {
                HStack {
                    Text("Study All")
                    Spacer()
                    Image(systemName: "arrow.right")
                }
                .padding()
                .background(.purple.opacity(0.1))
                .foregroundStyle(.purple)
                .clipShape(RoundedRectangle(cornerRadius: 10))
            }
        }
        .sheet(isPresented: $showingAllCards) {
            FlashcardStudyView(flashcards: flashcards)
        }
    }
}

// MARK: - Quiz View
struct QuizView: View {
    @Environment(DataManager.self) private var dataManager
    @State private var selectedQuiz: Quiz?
    
    var body: some View {
        NavigationStack {
            Group {
                if dataManager.quizzes.isEmpty {
                    ContentUnavailableView(
                        "No quizzes available",
                        systemImage: "questionmark.circle",
                        description: Text("Generate quizzes from your study materials")
                    )
                } else {
                    List {
                        ForEach(dataManager.quizzes) { quiz in
                            QuizRow(quiz: quiz)
                                .onTapGesture {
                                    selectedQuiz = quiz
                                }
                        }
                    }
                }
            }
            .navigationTitle("Quiz")
            .sheet(item: $selectedQuiz) { quiz in
                QuizTakingView(quiz: quiz)
            }
        }
    }
}

struct QuizRow: View {
    let quiz: Quiz
    
    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            HStack {
                Text(quiz.title)
                    .font(.headline)
                Spacer()
                if quiz.completed {
                    Image(systemName: "checkmark.circle.fill")
                        .foregroundStyle(.green)
                }
            }
            
            HStack {
                Label("\(quiz.questions.count) questions", systemImage: "list.bullet")
                    .font(.caption)
                    .foregroundStyle(.secondary)
                
                Spacer()
                
                if let score = quiz.score {
                    Text("\(Int(score))%")
                        .font(.caption)
                        .fontWeight(.semibold)
                        .padding(.horizontal, 8)
                        .padding(.vertical, 4)
                        .background(scoreColor(score).opacity(0.2))
                        .foregroundStyle(scoreColor(score))
                        .clipShape(Capsule())
                }
            }
        }
        .padding(.vertical, 4)
    }
    
    private func scoreColor(_ score: Double) -> Color {
        if score >= 80 { return .green }
        if score >= 60 { return .orange }
        return .red
    }
}

// MARK: - Progress Dashboard View
struct ProgressDashboardView: View {
    @Environment(DataManager.self) private var dataManager
    
    var body: some View {
        NavigationStack {
            ScrollView {
                VStack(spacing: 20) {
                    // Study Streak
                    VStack(spacing: 8) {
                        Text("ðŸ”¥")
                            .font(.system(size: 50))
                        Text("\(dataManager.progress.currentStreak) Day Streak")
                            .font(.title2)
                            .fontWeight(.semibold)
                        if dataManager.progress.currentStreak > 0 {
                            Text("Keep it up! Your longest streak: \(dataManager.progress.longestStreak) days")
                                .font(.caption)
                                .foregroundStyle(.secondary)
                        } else {
                            Text("Start learning to build your streak!")
                                .font(.caption)
                                .foregroundStyle(.secondary)
                        }
                    }
                    .frame(maxWidth: .infinity)
                    .padding()
                    .background(.orange.opacity(0.1))
                    .clipShape(RoundedRectangle(cornerRadius: 12))
                    .padding(.horizontal)
                    
                    // Stats Grid
                    LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible())], spacing: 12) {
                        StatCard(
                            value: "\(dataManager.documents.count)",
                            label: "Documents",
                            icon: "doc.fill",
                            color: .blue
                        )
                        StatCard(
                            value: "\(dataManager.flashcards.count)",
                            label: "Flashcards",
                            icon: "rectangle.stack.fill",
                            color: .purple
                        )
                        StatCard(
                            value: "\(dataManager.quizzes.count)",
                            label: "Quizzes",
                            icon: "pencil.circle.fill",
                            color: .green
                        )
                        StatCard(
                            value: String(format: "%.0f%%", dataManager.progress.overallAccuracy),
                            label: "Accuracy",
                            icon: "target",
                            color: .red
                        )
                    }
                    .padding(.horizontal)
                    
                    // Weak Topics
                    if !dataManager.weakTopics.isEmpty {
                        VStack(alignment: .leading, spacing: 12) {
                            Text("Areas to Improve")
                                .font(.headline)
                                .padding(.horizontal)
                            
                            ForEach(dataManager.weakTopics.prefix(5)) { topic in
                                HStack {
                                    VStack(alignment: .leading, spacing: 4) {
                                        Text(topic.topic)
                                            .font(.subheadline)
                                            .fontWeight(.medium)
                                        Text("\(topic.incorrectCount) mistakes")
                                            .font(.caption)
                                            .foregroundStyle(.secondary)
                                    }
                                    Spacer()
                                    Image(systemName: "arrow.right")
                                        .foregroundStyle(.tertiary)
                                }
                                .padding()
                                .background(Color(.systemBackground))
                                .clipShape(RoundedRectangle(cornerRadius: 10))
                            }
                            .padding(.horizontal)
                        }
                    }
                }
                .padding(.vertical)
            }
            .navigationTitle("Progress")
            .onAppear {
                dataManager.analyzeWeakTopics()
            }
        }
    }
}

// MARK: - Stat Card
struct StatCard: View {
    let value: String
    let label: String
    let icon: String
    let color: Color
    
    var body: some View {
        VStack(spacing: 8) {
            Image(systemName: icon)
                .foregroundStyle(color)
            Text(value)
                .font(.title2)
                .fontWeight(.bold)
            Text(label)
                .font(.caption)
                .foregroundStyle(.secondary)
        }
        .frame(maxWidth: .infinity)
        .padding()
        .background(color.opacity(0.1))
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

#Preview {
    ContentView()
}
```

### cognify/FlashcardStudyView.swift
Create file `FlashcardStudyView.swift` inside `cognify` folder.
```swift
//
//  FlashcardStudyView.swift
//  cognify
//
//  Created by Kusalab Dewan on 23/3/2026.
//

import SwiftUI

struct FlashcardStudyView: View {
    @EnvironmentObject var dataManager: DataManager
    @Environment(\.dismiss) private var dismiss
    let flashcards: [Flashcard]
    
    @State private var currentIndex = 0
    @State private var isShowingAnswer = false
    @State private var offset: CGSize = .zero
    @State private var rotation: Double = 0
    @State private var correctCount = 0
    @State private var incorrectCount = 0
    @State private var isComplete = false
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 20) {
                if flashcards.isEmpty {
                    ContentUnavailableView(
                        "No Flashcards",
                        systemImage: "rectangle.stack",
                        description: Text("Add some flashcards to study")
                    )
                } else if isComplete {
                    // Completion View
                    VStack(spacing: 20) {
                        Image(systemName: "checkmark.circle.fill")
                            .font(.system(size: 80))
                            .foregroundStyle(.green)
                        
                        Text("Study Session Complete!")
                            .font(.title)
                            .fontWeight(.bold)
                        
                        VStack(spacing: 12) {
                            HStack {
                                Label("\(correctCount)", systemImage: "checkmark.circle.fill")
                                    .foregroundStyle(.green)
                                Spacer()
                                Text("Correct")
                            }
                            
                            HStack {
                                Label("\(incorrectCount)", systemImage: "xmark.circle.fill")
                                    .foregroundStyle(.red)
                                Spacer()
                                Text("Incorrect")
                            }
                            
                            Divider()
                            
                            HStack {
                                Text("Accuracy")
                                Spacer()
                                Text("\(Int(Double(correctCount) / Double(correctCount + incorrectCount) * 100))%")
                                    .fontWeight(.bold)
                            }
                        }
                        .padding()
                        .background(Color(.systemGray6))
                        .clipShape(RoundedRectangle(cornerRadius: 12))
                        .padding(.horizontal)
                        
                        Button("Done") {
                            dismiss()
                        }
                        .buttonStyle(.borderedProminent)
                        .padding()
                    }
                } else {
                    // Progress
                    VStack(spacing: 8) {
                        HStack {
                            Text("Card \(currentIndex + 1) of \(flashcards.count)")
                                .font(.headline)
                            Spacer()
                            HStack(spacing: 16) {
                                Label("\(correctCount)", systemImage: "checkmark.circle.fill")
                                    .foregroundStyle(.green)
                                Label("\(incorrectCount)", systemImage: "xmark.circle.fill")
                                    .foregroundStyle(.red)
                            }
                            .font(.caption)
                        }
                        
                        ProgressView(value: Double(currentIndex), total: Double(flashcards.count))
                    }
                    .padding()
                    
                    // Flashcard
                    ZStack {
                        ForEach(Array(flashcards.enumerated()), id: \.element.id) { index, flashcard in
                            if index == currentIndex {
                                FlashcardView(
                                    flashcard: flashcard,
                                    isShowingAnswer: isShowingAnswer
                                )
                                .offset(offset)
                                .rotationEffect(.degrees(rotation))
                                .gesture(
                                    DragGesture()
                                        .onChanged { gesture in
                                            offset = gesture.translation
                                            rotation = Double(gesture.translation.width / 20)
                                        }
                                        .onEnded { gesture in
                                            let swipeThreshold: CGFloat = 100
                                            
                                            if gesture.translation.width > swipeThreshold {
                                                // Swiped right - Correct
                                                handleAnswer(correct: true)
                                            } else if gesture.translation.width < -swipeThreshold {
                                                // Swiped left - Incorrect
                                                handleAnswer(correct: false)
                                            } else {
                                                // Return to center
                                                withAnimation(.spring()) {
                                                    offset = .zero
                                                    rotation = 0
                                                }
                                            }
                                        }
                                )
                                .onTapGesture {
                                    withAnimation(.spring(response: 0.6, dampingFraction: 0.8)) {
                                        isShowingAnswer.toggle()
                                    }
                                }
                            }
                        }
                    }
                    .frame(maxWidth: .infinity, maxHeight: .infinity)
                    .padding()
                    
                    // Answer Buttons
                    if isShowingAnswer {
                        HStack(spacing: 20) {
                            Button(action: { handleAnswer(correct: false) }) {
                                Label("Incorrect", systemImage: "xmark.circle.fill")
                                    .frame(maxWidth: .infinity)
                            }
                            .buttonStyle(.bordered)
                            .tint(.red)
                            
                            Button(action: { handleAnswer(correct: true) }) {
                                Label("Correct", systemImage: "checkmark.circle.fill")
                                    .frame(maxWidth: .infinity)
                            }
                            .buttonStyle(.borderedProminent)
                            .tint(.green)
                        }
                        .padding()
                        .transition(.move(edge: .bottom).combined(with: .opacity))
                    }
                }
            }
            .navigationTitle("Study Session")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") {
                        dismiss()
                    }
                }
            }
        }
    }
    
    private func handleAnswer(correct: Bool) {
        let flashcard = flashcards[currentIndex]
        
        // Update data
        dataManager.updateFlashcard(flashcard, correct: correct)
        dataManager.recordStudySession()
        
        if correct {
            correctCount += 1
        } else {
            incorrectCount += 1
        }
        
        // Animate card away
        withAnimation(.easeOut(duration: 0.3)) {
            offset = correct ? CGSize(width: 500, height: 0) : CGSize(width: -500, height: 0)
            rotation = correct ? 15 : -15
        }
        
        // Move to next card or complete
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.3) {
            if currentIndex < flashcards.count - 1 {
                currentIndex += 1
                isShowingAnswer = false
                offset = .zero
                rotation = 0
            } else {
                isComplete = true
            }
        }
    }
}

// MARK: - Flashcard View
struct FlashcardView: View {
    let flashcard: Flashcard
    let isShowingAnswer: Bool
    
    var body: some View {
        ZStack {
            // Back (Answer)
            VStack(spacing: 16) {
                Text("Answer")
                    .font(.caption)
                    .foregroundStyle(.secondary)
                
                Spacer()
                
                Text(flashcard.answer)
                    .font(.title3)
                    .multilineTextAlignment(.center)
                
                Spacer()
                
                // Difficulty Badge
                HStack {
                    Label(flashcard.difficulty.rawValue, systemImage: "chart.bar.fill")
                        .font(.caption)
                        .padding(.horizontal, 12)
                        .padding(.vertical, 6)
                        .background(difficultyColor(flashcard.difficulty).opacity(0.2))
                        .foregroundStyle(difficultyColor(flashcard.difficulty))
                        .clipShape(Capsule())
                    
                    Spacer()
                    
                    if flashcard.correctCount + flashcard.incorrectCount > 0 {
                        Text("\(Int(flashcard.accuracy))% accuracy")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                    }
                }
            }
            .padding(24)
            .frame(maxWidth: .infinity, maxHeight: .infinity)
            .background(
                LinearGradient(
                    colors: [Color.blue.opacity(0.1), Color.purple.opacity(0.1)],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                )
            )
            .clipShape(RoundedRectangle(cornerRadius: 20))
            .shadow(color: .black.opacity(0.1), radius: 10, y: 5)
            .opacity(isShowingAnswer ? 1 : 0)
            .rotation3DEffect(
                .degrees(isShowingAnswer ? 0 : 180),
                axis: (x: 0, y: 1, z: 0)
            )
            
            // Front (Question)
            VStack(spacing: 16) {
                Text("Question")
                    .font(.caption)
                    .foregroundStyle(.secondary)
                
                Spacer()
                
                Text(flashcard.question)
                    .font(.title2)
                    .fontWeight(.medium)
                    .multilineTextAlignment(.center)
                
                Spacer()
                
                Text("Tap to reveal answer")
                    .font(.caption)
                    .foregroundStyle(.tertiary)
            }
            .padding(24)
            .frame(maxWidth: .infinity, maxHeight: .infinity)
            .background(
                LinearGradient(
                    colors: [Color.orange.opacity(0.1), Color.pink.opacity(0.1)],
                    startPoint: .topLeading,
                    endPoint: .bottomTrailing
                )
            )
            .clipShape(RoundedRectangle(cornerRadius: 20))
            .shadow(color: .black.opacity(0.1), radius: 10, y: 5)
            .opacity(isShowingAnswer ? 0 : 1)
            .rotation3DEffect(
                .degrees(isShowingAnswer ? -180 : 0),
                axis: (x: 0, y: 1, z: 0)
            )
        }
    }
    
    private func difficultyColor(_ difficulty: Difficulty) -> Color {
        switch difficulty {
        case .easy: return .green
        case .medium: return .orange
        case .hard: return .red
        }
    }
}

#Preview {
    FlashcardStudyView(flashcards: [
        Flashcard(question: "What is SwiftUI?", answer: "A declarative framework for building user interfaces across Apple platforms", documentId: UUID()),
        Flashcard(question: "What is a View in SwiftUI?", answer: "A protocol that defines a piece of user interface", documentId: UUID())
    ])
    .environmentObject(DataManager.shared)
}
```

### cognify/QuizTakingView.swift
Create file `QuizTakingView.swift` inside `cognify` folder.
```swift
//
//  QuizTakingView.swift
//  cognify
//
//  Created by Kusalab Dewan on 23/3/2026.
//

import SwiftUI

struct QuizTakingView: View {
    @EnvironmentObject var dataManager: DataManager
    @Environment(\.dismiss) private var dismiss
    
    @State private var quiz: Quiz
    @State private var currentQuestionIndex = 0
    @State private var selectedAnswer: String?
    @State private var showingResult = false
    @State private var isComplete = false
    
    init(quiz: Quiz) {
        _quiz = State(initialValue: quiz)
    }
    
    var currentQuestion: QuizQuestion {
        quiz.questions[currentQuestionIndex]
    }
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                if isComplete {
                    // Results View
                    ScrollView {
                        VStack(spacing: 24) {
                            // Score Header
                            VStack(spacing: 12) {
                                Image(systemName: scoreIcon)
                                    .font(.system(size: 60))
                                    .foregroundStyle(scoreColor)
                                
                                Text("\(Int(quiz.score ?? 0))%")
                                    .font(.system(size: 48, weight: .bold))
                                
                                Text(scoreMessage)
                                    .font(.title3)
                                    .foregroundStyle(.secondary)
                            }
                            .padding()
                            
                            // Summary
                            VStack(spacing: 12) {
                                HStack {
                                    Text("Correct")
                                    Spacer()
                                    Text("\(correctAnswersCount) / \(quiz.questions.count)")
                                        .fontWeight(.semibold)
                                }
                                
                                HStack {
                                    Text("Score")
                                    Spacer()
                                    Text("\(Int(quiz.score ?? 0))%")
                                        .fontWeight(.semibold)
                                }
                            }
                            .padding()
                            .background(Color(.systemGray6))
                            .clipShape(RoundedRectangle(cornerRadius: 12))
                            .padding(.horizontal)
                            
                            // Question Review
                            VStack(alignment: .leading, spacing: 16) {
                                Text("Review")
                                    .font(.headline)
                                    .padding(.horizontal)
                                
                                ForEach(Array(quiz.questions.enumerated()), id: \.element.id) { index, question in
                                    QuestionReviewCard(question: question, number: index + 1)
                                        .padding(.horizontal)
                                }
                            }
                            
                            Button("Done") {
                                dismiss()
                            }
                            .buttonStyle(.borderedProminent)
                            .padding()
                        }
                        .padding(.vertical)
                    }
                } else {
                    // Quiz Taking View
                    VStack(spacing: 0) {
                        // Progress Header
                        VStack(spacing: 12) {
                            HStack {
                                Text("Question \(currentQuestionIndex + 1) of \(quiz.questions.count)")
                                    .font(.headline)
                                Spacer()
                                Text("\(answeredCount) answered")
                                    .font(.caption)
                                    .foregroundStyle(.secondary)
                            }
                            
                            ProgressView(value: Double(currentQuestionIndex + 1), total: Double(quiz.questions.count))
                        }
                        .padding()
                        .background(Color(.systemBackground))
                        
                        Divider()
                        
                        ScrollView {
                            VStack(alignment: .leading, spacing: 24) {
                                // Question
                                Text(currentQuestion.question)
                                    .font(.title3)
                                    .fontWeight(.medium)
                                    .padding()
                                
                                // Options
                                VStack(spacing: 12) {
                                    ForEach(currentQuestion.options, id: \.self) { option in
                                        OptionButton(
                                            option: option,
                                            isSelected: selectedAnswer == option,
                                            isCorrect: showingResult ? option == currentQuestion.correctAnswer : nil,
                                            isWrong: showingResult ? (selectedAnswer == option && option != currentQuestion.correctAnswer) : false
                                        ) {
                                            if !showingResult {
                                                selectedAnswer = option
                                            }
                                        }
                                    }
                                }
                                .padding(.horizontal)
                                
                                // Explanation (shown after answer)
                                if showingResult, let explanation = currentQuestion.explanation {
                                    VStack(alignment: .leading, spacing: 8) {
                                        Label("Explanation", systemImage: "info.circle.fill")
                                            .font(.headline)
                                            .foregroundStyle(.blue)
                                        
                                        Text(explanation)
                                            .font(.body)
                                    }
                                    .padding()
                                    .background(Color.blue.opacity(0.1))
                                    .clipShape(RoundedRectangle(cornerRadius: 12))
                                    .padding(.horizontal)
                                }
                            }
                            .padding(.vertical)
                        }
                        
                        // Action Button
                        VStack {
                            Divider()
                            
                            if showingResult {
                                Button(action: nextQuestion) {
                                    Text(currentQuestionIndex < quiz.questions.count - 1 ? "Next Question" : "Finish Quiz")
                                        .font(.headline)
                                        .frame(maxWidth: .infinity)
                                }
                                .buttonStyle(.borderedProminent)
                                .padding()
                            } else {
                                Button(action: submitAnswer) {
                                    Text("Submit Answer")
                                        .font(.headline)
                                        .frame(maxWidth: .infinity)
                                }
                                .buttonStyle(.borderedProminent)
                                .disabled(selectedAnswer == nil)
                                .padding()
                            }
                        }
                        .background(Color(.systemBackground))
                    }
                }
            }
            .navigationTitle(quiz.title)
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                if !isComplete {
                    ToolbarItem(placement: .cancellationAction) {
                        Button("Cancel") {
                            dismiss()
                        }
                    }
                }
            }
        }
    }
    
    private var answeredCount: Int {
        quiz.questions.filter { $0.userAnswer != nil }.count
    }
    
    private var correctAnswersCount: Int {
        quiz.questions.filter { $0.isCorrect == true }.count
    }
    
    private var scoreIcon: String {
        guard let score = quiz.score else { return "questionmark.circle" }
        if score >= 80 { return "star.fill" }
        if score >= 60 { return "hand.thumbsup.fill" }
        return "arrow.clockwise"
    }
    
    private var scoreColor: Color {
        guard let score = quiz.score else { return .gray }
        if score >= 80 { return .green }
        if score >= 60 { return .orange }
        return .red
    }
    
    private var scoreMessage: String {
        guard let score = quiz.score else { return "Let's begin!" }
        if score >= 80 { return "Excellent work!" }
        if score >= 60 { return "Good job!" }
        return "Keep practicing!"
    }
    
    private func submitAnswer() {
        guard let answer = selectedAnswer else { return }
        
        // Save user's answer
        quiz.questions[currentQuestionIndex].userAnswer = answer
        
        withAnimation {
            showingResult = true
        }
    }
    
    private func nextQuestion() {
        if currentQuestionIndex < quiz.questions.count - 1 {
            currentQuestionIndex += 1
            selectedAnswer = quiz.questions[currentQuestionIndex].userAnswer
            showingResult = selectedAnswer != nil
        } else {
            completeQuiz()
        }
    }
    
    private func completeQuiz() {
        // Calculate score
        let correct = correctAnswersCount
        let total = quiz.questions.count
        let score = Double(correct) / Double(total) * 100
        
        quiz.score = score
        quiz.completed = true
        
        // Update in data manager
        dataManager.updateQuiz(quiz)
        dataManager.recordStudySession()
        
        withAnimation {
            isComplete = true
        }
    }
}

// MARK: - Option Button
struct OptionButton: View {
    let option: String
    let isSelected: Bool
    let isCorrect: Bool?
    let isWrong: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack {
                Text(option)
                    .multilineTextAlignment(.leading)
                    .foregroundStyle(textColor)
                
                Spacer()
                
                if let isCorrect = isCorrect {
                    Image(systemName: isCorrect ? "checkmark.circle.fill" : (isWrong ? "xmark.circle.fill" : ""))
                        .foregroundStyle(isCorrect ? .green : .red)
                }
            }
            .padding()
            .background(backgroundColor)
            .overlay(
                RoundedRectangle(cornerRadius: 12)
                    .stroke(borderColor, lineWidth: 2)
            )
            .clipShape(RoundedRectangle(cornerRadius: 12))
        }
        .buttonStyle(.plain)
    }
    
    private var backgroundColor: Color {
        if let isCorrect = isCorrect {
            return isCorrect ? Color.green.opacity(0.1) : (isWrong ? Color.red.opacity(0.1) : Color(.systemGray6))
        }
        return isSelected ? Color.blue.opacity(0.1) : Color(.systemGray6)
    }
    
    private var borderColor: Color {
        if let isCorrect = isCorrect {
            return isCorrect ? .green : (isWrong ? .red : .clear)
        }
        return isSelected ? .blue : .clear
    }
    
    private var textColor: Color {
        if isCorrect == true {
            return .green
        } else if isWrong {
            return .red
        }
        return .primary
    }
}

// MARK: - Question Review Card
struct QuestionReviewCard: View {
    let question: QuizQuestion
    let number: Int
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            HStack {
                Text("Q\(number)")
                    .font(.caption)
                    .fontWeight(.semibold)
                    .foregroundStyle(.white)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(question.isCorrect == true ? Color.green : Color.red)
                    .clipShape(Capsule())
                
                Spacer()
                
                Image(systemName: question.isCorrect == true ? "checkmark.circle.fill" : "xmark.circle.fill")
                    .foregroundStyle(question.isCorrect == true ? .green : .red)
            }
            
            Text(question.question)
                .font(.subheadline)
                .fontWeight(.medium)
            
            VStack(alignment: .leading, spacing: 4) {
                HStack {
                    Text("Your answer:")
                        .font(.caption)
                        .foregroundStyle(.secondary)
                    Text(question.userAnswer ?? "No answer")
                        .font(.caption)
                        .fontWeight(.medium)
                }
                
                if question.isCorrect == false {
                    HStack {
                        Text("Correct answer:")
                            .font(.caption)
                            .foregroundStyle(.secondary)
                        Text(question.correctAnswer)
                            .font(.caption)
                            .fontWeight(.medium)
                            .foregroundStyle(.green)
                    }
                }
            }
        }
        .padding()
        .background(Color(.systemGray6))
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

#Preview {
    QuizTakingView(quiz: Quiz(
        title: "Sample Quiz",
        documentId: UUID(),
        questions: [
            QuizQuestion(
                question: "What is SwiftUI?",
                options: ["A framework", "A language", "A tool", "An IDE"],
                correctAnswer: "A framework",
                explanation: "SwiftUI is Apple's declarative framework for building user interfaces."
            ),
            QuizQuestion(
                question: "Which keyword is used for asynchronous functions?",
                options: ["await", "async", "defer", "task"],
                correctAnswer: "async"
            )
        ]
    ))
    .environmentObject(DataManager.shared)
}
```

### cognify/SupportingViews.swift
Create file `SupportingViews.swift` inside `cognify` folder.
```swift
//
//  SupportingViews.swift
//  cognify
//
//  Created by Kusalab Dewan on 23/3/2026.
//

import SwiftUI
import UIKit

// MARK: - Camera View
struct CameraView: UIViewControllerRepresentable {
    let completion: (UIImage) -> Void
    @Environment(\.dismiss) private var dismiss
    
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.sourceType = .camera
        picker.delegate = context.coordinator
        return picker
    }
    
    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let parent: CameraView
        
        init(_ parent: CameraView) {
            self.parent = parent
        }
        
        func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey : Any]) {
            if let image = info[.originalImage] as? UIImage {
                parent.completion(image)
            }
            parent.dismiss()
        }
        
        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            parent.dismiss()
        }
    }
}

// MARK: - Text Input View
struct TextInputView: View {
    @Environment(\.dismiss) private var dismiss
    @State private var text = ""
    let completion: (String) -> Void
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 16) {
                TextEditor(text: $text)
                    .padding(8)
                    .background(Color(.systemGray6))
                    .clipShape(RoundedRectangle(cornerRadius: 10))
                    .frame(minHeight: 200)
                    .padding()
                
                Spacer()
            }
            .navigationTitle("Enter Text")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Cancel") {
                        dismiss()
                    }
                }
                ToolbarItem(placement: .confirmationAction) {
                    Button("Done") {
                        completion(text)
                        dismiss()
                    }
                    .disabled(text.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty)
                }
            }
        }
    }
}

// MARK: - Document Detail View
struct DocumentDetailView: View {
    @EnvironmentObject var dataManager: DataManager
    @Environment(\.dismiss) private var dismiss
    let document: Document
    @State private var showingDeleteConfirmation = false
    @State private var showingExplanation = false
    @State private var selectedExplanationLevel: ExplanationLevel = .detailed
    @State private var explanation: String?
    @State private var isGeneratingExplanation = false
    
    var body: some View {
        ScrollView {
            VStack(alignment: .leading, spacing: 20) {
                // Document Info
                VStack(alignment: .leading, spacing: 8) {
                    HStack {
                        Label(document.type.rawValue, systemImage: iconForType(document.type))
                            .font(.caption)
                            .padding(.horizontal, 12)
                            .padding(.vertical, 6)
                            .background(.blue.opacity(0.1))
                            .foregroundStyle(.blue)
                            .clipShape(Capsule())
                        
                        Spacer()
                        
                        Text(document.createdAt, style: .date)
                            .font(.caption)
                            .foregroundStyle(.secondary)
                    }
                }
                .padding(.horizontal)
                
                // Content
                VStack(alignment: .leading, spacing: 8) {
                    Text("Content")
                        .font(.headline)
                    Text(document.content)
                        .font(.body)
                        .textSelection(.enabled)
                }
                .padding()
                .background(Color(.systemGray6))
                .clipShape(RoundedRectangle(cornerRadius: 12))
                .padding(.horizontal)
                
                // AI Actions
                VStack(spacing: 12) {
                    Button(action: { showingExplanation = true }) {
                        Label("Get AI Explanation", systemImage: "brain")
                            .frame(maxWidth: .infinity)
                    }
                    .buttonStyle(.borderedProminent)
                    
                    let flashcards = dataManager.getFlashcards(for: document.id)
                    NavigationLink(destination: FlashcardStudyView(flashcards: flashcards)) {
                        HStack {
                            Label("Study Flashcards (\(flashcards.count))", systemImage: "rectangle.stack")
                            Spacer()
                        }
                        .frame(maxWidth: .infinity)
                        .padding()
                        .background(.purple.opacity(0.1))
                        .foregroundStyle(.purple)
                        .clipShape(RoundedRectangle(cornerRadius: 10))
                    }
                    .disabled(flashcards.isEmpty)
                    
                    let quizzes = dataManager.getQuizzes(for: document.id)
                    if let quiz = quizzes.first {
                        NavigationLink(destination: QuizTakingView(quiz: quiz)) {
                            HStack {
                                Label("Take Quiz", systemImage: "questionmark.circle")
                                Spacer()
                            }
                            .frame(maxWidth: .infinity)
                            .padding()
                            .background(.green.opacity(0.1))
                            .foregroundStyle(.green)
                            .clipShape(RoundedRectangle(cornerRadius: 10))
                        }
                    }
                }
                .padding(.horizontal)
                
                // Delete Button
                Button(role: .destructive, action: { showingDeleteConfirmation = true }) {
                    Label("Delete Document", systemImage: "trash")
                        .frame(maxWidth: .infinity)
                }
                .buttonStyle(.bordered)
                .padding(.horizontal)
            }
            .padding(.vertical)
        }
        .navigationTitle(document.title)
        .navigationBarTitleDisplayMode(.inline)
        .sheet(isPresented: $showingExplanation) {
            ExplanationView(
                document: document,
                selectedLevel: $selectedExplanationLevel
            )
        }
        .alert("Delete Document?", isPresented: $showingDeleteConfirmation) {
            Button("Delete", role: .destructive) {
                dataManager.deleteDocument(document)
                dismiss()
            }
            Button("Cancel", role: .cancel) {}
        } message: {
            Text("This will also delete all associated flashcards and quizzes.")
        }
    }
    
    private func iconForType(_ type: DocumentType) -> String {
        switch type {
        case .pdf: return "doc.fill"
        case .image: return "photo.fill"
        case .text: return "doc.text.fill"
        }
    }
}

// MARK: - Explanation View
struct ExplanationView: View {
    @Environment(\.dismiss) private var dismiss
    let document: Document
    @Binding var selectedLevel: ExplanationLevel
    @State private var explanation: String?
    @State private var isLoading = false
    @State private var errorMessage: String?
    
    var body: some View {
        NavigationStack {
            VStack(spacing: 16) {
                Picker("Level", selection: $selectedLevel) {
                    Text("Beginner").tag(ExplanationLevel.beginner)
                    Text("Detailed").tag(ExplanationLevel.detailed)
                    Text("Exam-Focused").tag(ExplanationLevel.examFocused)
                }
                .pickerStyle(.segmented)
                .padding()
                .onChange(of: selectedLevel) { oldValue, newValue in
                    generateExplanation()
                }
                
                ScrollView {
                    if isLoading {
                        ProgressView("Generating explanation...")
                            .padding()
                    } else if let explanation = explanation {
                        Text(explanation)
                            .padding()
                            .textSelection(.enabled)
                    } else if let error = errorMessage {
                        VStack(spacing: 12) {
                            Image(systemName: "exclamationmark.triangle")
                                .font(.largeTitle)
                                .foregroundStyle(.orange)
                            Text(error)
                                .foregroundStyle(.secondary)
                            Button("Try Again") {
                                generateExplanation()
                            }
                            .buttonStyle(.bordered)
                        }
                        .padding()
                    } else {
                        Text("Tap 'Generate' to get an AI explanation")
                            .foregroundStyle(.secondary)
                            .padding()
                    }
                }
            }
            .navigationTitle("AI Explanation")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .cancellationAction) {
                    Button("Done") { dismiss() }
                }
                ToolbarItem(placement: .primaryAction) {
                    Button("Generate") {
                        generateExplanation()
                    }
                    .disabled(isLoading)
                }
            }
            .onAppear {
                if explanation == nil {
                    generateExplanation()
                }
            }
        }
    }
    
    private func generateExplanation() {
        isLoading = true
        errorMessage = nil
        
        Task {
            do {
                let result: String
                if NetworkService.USE_MOCK_DATA {
                    result = try await MockDataService.shared.generateExplanation(
                        from: document.content,
                        level: selectedLevel
                    )
                } else {
                    result = try await NetworkService.shared.generateExplanation(
                        from: document.content,
                        level: selectedLevel
                    )
                }
                explanation = result
                isLoading = false
            } catch {
                errorMessage = error.localizedDescription
                isLoading = false
            }
        }
    }
}
```

## Git Steps by Feature
After `cognifyApp.swift`:
```bash
git add cognify/cognifyApp.swift
git commit -m "feat(ui): add app entry point"
git push origin master
```

After `ContentView.swift`:
```bash
git add cognify/ContentView.swift
git commit -m "feat(ui): add main navigation and dashboard"
git push origin master
```

After `FlashcardStudyView.swift`:
```bash
git add cognify/FlashcardStudyView.swift
git commit -m "feat(ui): add flashcard study flow"
git push origin master
```

After `QuizTakingView.swift`:
```bash
git add cognify/QuizTakingView.swift
git commit -m "feat(ui): add quiz taking flow"
git push origin master
```

After `SupportingViews.swift`:
```bash
git add cognify/SupportingViews.swift
git commit -m "feat(ui): add supporting study views"
git push origin master
```
