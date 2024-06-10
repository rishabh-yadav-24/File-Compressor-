#include <iostream>
#include <fstream>
#include <queue>
#include <unordered_map>
#include <vector>
#include <bitset>

using namespace std;

// A Node structure for Huffman Tree nodes
struct Node {
    char ch;
    int freq;
    Node *left, *right;

    Node(char ch, int freq) : ch(ch), freq(freq), left(nullptr), right(nullptr) {}
};

// Comparison function for priority queue
struct Compare {
    bool operator()(Node* left, Node* right) {
        return left->freq > right->freq;
    }
};

// Function to build the Huffman Tree
Node* buildHuffmanTree(const string& text) {
    unordered_map<char, int> freq_map;
    for (char ch : text) {
        freq_map[ch]++;
    }

    priority_queue<Node*, vector<Node*>, Compare> pq;
    for (auto& pair : freq_map) {
        pq.push(new Node(pair.first, pair.second));
    }

    while (pq.size() > 1) {
        Node* left = pq.top(); pq.pop();
        Node* right = pq.top(); pq.pop();
        Node* merged = new Node('\0', left->freq + right->freq);
        merged->left = left;
        merged->right = right;
        pq.push(merged);
    }

    return pq.top();
}

// Function to generate Huffman codes
void generateCodes(Node* node, const string& prefix, unordered_map<char, string>& code_map) {
    if (!node) return;

    if (node->ch != '\0') {
        code_map[node->ch] = prefix;
    } else {
        generateCodes(node->left, prefix + "0", code_map);
        generateCodes(node->right, prefix + "1", code_map);
    }
}

// Function to encode the text using Huffman codes
string encodeText(const string& text, const unordered_map<char, string>& code_map) {
    string encoded_text;
    for (char ch : text) {
        encoded_text += code_map.at(ch);
    }
    return encoded_text;
}

// Function to pad the encoded text to make its length a multiple of 8
string padEncodedText(const string& encoded_text) {
    int extra_padding = 8 - encoded_text.size() % 8;
    string padded_info = bitset<8>(extra_padding).to_string();
    return padded_info + encoded_text + string(extra_padding, '0');
}

// Function to convert the padded encoded text to bytes
vector<uint8_t> getByteArray(const string& padded_encoded_text) {
    vector<uint8_t> byte_array;
    for (size_t i = 0; i < padded_encoded_text.size(); i += 8) {
        byte_array.push_back(bitset<8>(padded_encoded_text.substr(i, 8)).to_ulong());
    }
    return byte_array;
}

// Function to compress a file using Huffman coding
void compressFile(const string& input_path, const string& output_path) {
    ifstream input_file(input_path);
    string text((istreambuf_iterator<char>(input_file)), istreambuf_iterator<char>());

    Node* huffman_tree = buildHuffmanTree(text);
    unordered_map<char, string> code_map;
    generateCodes(huffman_tree, "", code_map);
    string encoded_text = encodeText(text, code_map);
    string padded_encoded_text = padEncodedText(encoded_text);
    vector<uint8_t> byte_array = getByteArray(padded_encoded_text);

    ofstream output_file(output_path, ios::binary);
    for (uint8_t byte : byte_array) {
        output_file.put(byte);
    }
}

// Function to remove padding from encoded text
string removePadding(const string& padded_encoded_text) {
    string padded_info = padded_encoded_text.substr(0, 8);
    int extra_padding = bitset<8>(padded_info).to_ulong();
    return padded_encoded_text.substr(8, padded_encoded_text.size() - 8 - extra_padding);
}

// Function to decode the encoded text using the Huffman tree
string decodeText(const string& encoded_text, Node* huffman_tree) {
    string decoded_text;
    Node* current_node = huffman_tree;
    for (char bit : encoded_text) {
        current_node = (bit == '0') ? current_node->left : current_node->right;
        if (!current_node->left && !current_node->right) {
            decoded_text += current_node->ch;
            current_node = huffman_tree;
        }
    }
    return decoded_text;
}

// Function to decompress a file
void decompressFile(const string& input_path, const string& output_path, Node* huffman_tree) {
    ifstream input_file(input_path, ios::binary);
    string bit_string;
    char byte;
    while (input_file.get(byte)) {
        bit_string += bitset<8>(byte).to_string();
    }

    string encoded_text = removePadding(bit_string);
    string decoded_text = decodeText(encoded_text, huffman_tree);

    ofstream output_file(output_path);
    output_file << decoded_text;
}

// Helper function to delete Huffman Tree
void deleteHuffmanTree(Node* node) {
    if (!node) return;
    deleteHuffmanTree(node->left);
    deleteHuffmanTree(node->right);
    delete node;
}

// Example usage
int main() {
    string input_path = "input.txt";
    string compressed_path = "compressed.bin";
    string output_path = "output.txt";

    // Compressing a file
    compressFile(input_path, compressed_path);

    // Building Huffman Tree again for decompression (in real applications, the tree should be stored/retrieved)
    ifstream input_file(input_path);
    string text((istreambuf_iterator<char>(input_file)), istreambuf_iterator<char>());
    Node* huffman_tree = buildHuffmanTree(text);

    // Decompressing a file
    decompressFile(compressed_path, output_path, huffman_tree);

    // Clean up
    deleteHuffmanTree(huffman_tree);

    return 0;
}
