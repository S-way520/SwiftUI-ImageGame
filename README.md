# SwiftUI-ImageGame
A simple and fun puzzle game
# The first part
https://github.com/S-way520/SwiftUI-ImageGame/assets/95877651/3d1b80f6-6117-458b-83ca-064e665dddb1
# The second part
Code file 1
```swift
import SwiftUI
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```
Code file 2
```swift
import SwiftUI
struct ContentView: View {
    @State private var tiles: [ImagePiece] = ImagePiece.createShuffledPieces(image: UIImage(imageLiteralResourceName: "myImage1"))
    @State private var steps: Int = 0
    @State private var imageIndex: Int = 1 // Initial image index
    var isWin: Bool {
        return tiles.enumerated().allSatisfy { $0.element.id == $0.offset }
    }
    var body: some View {
        VStack {
            HStack {
                Button("Refresh") { refreshPuzzle() }
                Spacer()
                Text("Steps: \(steps)")
            }
            LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 3), spacing: 10) {
                ForEach(0..<9, id: \.self) { index in
                    ImageTileView(piece: tiles[index])
                        .onTapGesture {
                            handleTap(index)
                        }
                }
            }
        }
        .padding()
        .alert(isPresented: .constant(isWin)) {
            Alert(
                title: Text("You Win!"),
                message: Text("Congratulations! You solved the puzzle in \(steps) steps."),
                dismissButton: .default(Text("OK"))
            )
        }
    }
    private func handleTap(_ tappedIndex: Int) {
        guard let emptyIndex = tiles.firstIndex(where: { $0.isEmpty }) else {
            return
        }
        withAnimation {
            tiles.swapAt(tappedIndex, emptyIndex)
            steps += 1
        }
    }
    private func refreshPuzzle() {
        // Increment the image index and load the corresponding image
        imageIndex = (imageIndex % 3) + 1
        let imageName = "myImage\(imageIndex)"
        guard let newImage = UIImage(named: imageName) else {
            fatalError("Image \(imageName) not found")
        }
        // Create new shuffled pieces using the new image
        tiles = ImagePiece.createShuffledPieces(image: newImage)
        steps = 0
    }
}
struct ImageTileView: View {
    let piece: ImagePiece
    var body: some View {
        Image(uiImage: piece.image)
            .resizable()
            .aspectRatio(contentMode: .fill)
            .cornerRadius(5)
            .overlay(
                RoundedRectangle(cornerRadius: 5, style: .continuous)
                    .stroke(.blue.opacity(piece.isEmpty ? 0.8 : 0), lineWidth: 2)
            )
    }
}
struct ImagePiece {
    let id: Int
    let image: UIImage
    var isEmpty: Bool
    static func createShuffledPieces(image: UIImage) -> [ImagePiece] {
        var pieces = (0..<9).map { index -> ImagePiece in
            let x = CGFloat(index % 3)
            let y = CGFloat(index / 3)
            let size: CGSize = CGSize(width: image.size.width / 3, height: image.size.height / 3)
            let rect = CGRect(x: x * size.width, y: y * size.height, width: size.width, height: size.height)
            let pieceImage = image.cropped(to: rect)
            return ImagePiece(id: index, image: pieceImage, isEmpty: false)
        }
        pieces.shuffle()
        if let lastIndex = pieces.firstIndex(where: { $0.id == 8 }) {
            pieces[lastIndex].isEmpty = true
        }
        return pieces
    }
}
extension UIImage {
    func cropped(to rect: CGRect) -> UIImage {
        guard let cgImage = self.cgImage?.cropping(to: rect) else {
            return self
        }
        return UIImage(cgImage: cgImage)
    }
}
```
