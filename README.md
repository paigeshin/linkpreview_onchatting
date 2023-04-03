# linkpreview_onchatting

```swift
//
//  ContentView.swift
//  LinkPreview
//
//  Created by paige shin on 2023/04/04.
//

import SwiftUI
import LinkPresentation

struct Message: Identifiable {
    var id = UUID()
    var message: String
    var date = Date()
    var previewLoading = false
    var linkMetaData: LPLinkMetadata?
    var linkURL: URL?
}

class MessageViewModel: ObservableObject {
    
    @Published var message: String = ""
    
    // All Messages...
    @Published var messages: [Message] = []
    
    func sendMessage() {
        
        // adding to messages
        if !self.isMessageURL() {
            self.addToMessage()
            return
        }
        
        // Extracting URL Meta Data...
        // Before that adding loading indicator...
        guard let url = URL(string: self.message) else {
            return
        }
        let urlMessage = Message(message: self.message, previewLoading: true, linkURL: url)
        self.messages.append(urlMessage)
        
        let provider = LPMetadataProvider()
        provider.startFetchingMetadata(for: url) { meta, err in
            DispatchQueue.main.async {
                if let _ = err {
                    self.addToMessage()
                    return
                }
                
                guard let meta else {
                    self.addToMessage()
                    return
                }
                
                
                if let index = self.messages.firstIndex(where: { message in
                    return urlMessage.id == message.id
                }) {
                    self.messages[index].linkMetaData = meta
                }
            }
            
        }
    }
    
    func isMessageURL() -> Bool {
        guard let url = URL(string: self.message) else {
           return false
        }
        return UIApplication.shared.canOpenURL(url)
    }
    
    func addToMessage() {
        self.messages.append(Message(message: self.message))
        self.message = ""
    }
    
}

struct ContentView: View {
    
    @StateObject var messageData: MessageViewModel = MessageViewModel()
    
    var body: some View {
        NavigationView {
            
            ScrollView(.vertical, showsIndicators: false) {
                
                VStack(spacing: 15) {
                    
                    ForEach(self.messageData.messages) { message in
                        CardView(message: message)
                    }
                    
                } //: VSTACK
                
            } //: SCROLL VIEW
            // Safe Area Bottom bar with Textfield....
            .safeAreaInset(edge: .bottom) {
                HStack(spacing: 12) {
                    TextField("Enter Message", text: self.$messageData.message)
                        .textFieldStyle(.roundedBorder)
                        .textCase(.lowercase)
                        .textInputAutocapitalization(.none)
                        .disableAutocorrection(true)
                    
                    // Send Button ...
                    Button {
                        self.messageData.sendMessage()
                    } label: {
                        Image(systemName: "paperplane")
                            .font(.title3)
                    }
                        
                } //: HSTACK
                .padding()
                .padding(.top)
                .background(.ultraThinMaterial)
            }
            .navigationTitle("Link Preview")
            // We always need dark mode...
            .preferredColorScheme(.dark)
            
        }
    }
    
    @ViewBuilder
    func CardView(message: Message) -> some View {
        Group {
            // if there is a link preview then showing it...
            // else showing loading...
            // otherwise showing general message...
            if message.previewLoading {

                Group {
                    if let metadata = message.linkMetaData {
                        LinkPreview(metadata: metadata)
                    } else {
                        HStack(spacing: 10) {
                            Text(message.linkURL?.host ?? "")
                                .font(.caption)

                            ProgressView()
                                .tint(.white)
                        }
                        .padding(.vertical, 10)
                        .padding(.horizontal)
                        .background(Color.gray.opacity(0.35))
                        .cornerRadius(10)
                        .frame(maxWidth: .infinity, alignment: .trailing)
                    }
                }

            } else {

                Text(message.message)
                    .padding(.vertical, 10)
                    .padding(.horizontal)
                    .foregroundColor(.white)
                    .background(Color.blue)
                    .cornerRadius(10)
                    .frame(width: UIScreen.main.bounds.width - 80, alignment: .trailing)
                    .frame(maxWidth: .infinity, alignment: .trailing)

            }
        }
    }
    
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

struct LinkPreview: UIViewRepresentable {
    
    var metadata: LPLinkMetadata
    
    func makeUIView(context: Context) -> LPLinkView {
        let preview = LPLinkView(metadata: self.metadata)
        return preview
    }
    
    func updateUIView(_ uiView: LPLinkView, context: Context) {
        uiView.metadata = self.metadata
    }
    
}

```
