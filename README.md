#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <algorithm>
#include <cctype>

using namespace std;

// Структура для хранения информации о слове
struct Word {
    string word;
    int frequency;
};

// Функция для загрузки словаря из файла
vector<Word> loadDictionary(const string& filename) {
    vector<Word> dictionary;
    ifstream file(filename);
    string word;

    while (getline(file, word)) {
        // Преобразуем слово в нижний регистр
        transform(word.begin(), word.end(), word.begin(), ::tolower);

        // Проверяем, есть ли слово в словаре
        auto it = find_if(dictionary.begin(), dictionary.end(),
                        [&word](const Word& dictWord) { return dictWord.word == word; });

        if (it != dictionary.end()) {
            // Если слово уже есть, увеличиваем его частоту
            it->frequency++;
        } else {
            // Если слова нет, добавляем его в словарь с частотой 1
            dictionary.push_back({word, 1});
        }
    }
    file.close();

    return dictionary;
}

// Функция для проверки слова на правильность
bool isWordCorrect(const string& word, const vector<Word>& dictionary) {
    return find_if(dictionary.begin(), dictionary.end(),
                    [&word](const Word& dictWord) { return dictWord.word == word; }) != dictionary.end();
}

// Функция для проверки текста на ошибки
vector<string> spellCheck(const string& text, const vector<Word>& dictionary) {
    vector<string> misspelledWords;
    string word;

    // Разделяем текст на слова
    for (size_t i = 0; i < text.size(); i++) {
        if (isalpha(text[i])) {
            word += tolower(text[i]);
        } else {
            if (!word.empty()) {
                if (!isWordCorrect(word, dictionary)) {
                    misspelledWords.push_back(word);
                }
                word.clear();
            }
        }
    }

    // Проверяем последнее слово
    if (!word.empty()) {
        if (!isWordCorrect(word, dictionary)) {
            misspelledWords.push_back(word);
        }
    }

    return misspelledWords;
}

int main() {
    string filename = "dictionary.txt"; // Имя файла со словарем
    vector<Word> dictionary = loadDictionary(filename);

    string text;
    cout << "Введите текст для проверки: ";
    getline(cin, text);

    vector<string> misspelledWords = spellCheck(text, dictionary);

    if (misspelledWords.empty()) {
        cout << "Ошибок не найдено." << endl;
    } else {
        cout << "Найдены следующие ошибки:" << endl;
        for (const string& word : misspelledWords) {
            cout << word << endl;
        }
    }

    return 0;
}
