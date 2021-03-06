#include <iostream>
#include <vector>
#include <initializer_list>
#include <memory>
#include <random>

template <class KeyType, class ValueType, class Hash = std::hash<KeyType>>
class HashMap {
public:
    int count;
    int filled;
    int capacity;
    Hash hash;
    std::vector<bool> used;
    std::vector<std::shared_ptr<std::pair<const KeyType, ValueType>>> table;

    class iterator {
    private:
        std::vector<bool>::iterator used_it, used_end_it;
        typename std::vector<std::shared_ptr<std::pair<const KeyType,
                ValueType>>>::iterator table_it;

        void next() {
            while (used_end_it != used_it && !*used_it) {
                used_it++;
                table_it++;
            }
        }

    public:
        iterator() {}

        iterator(std::vector<bool>::iterator used_it,
                 std::vector<bool>::iterator used_end_it,
                 typename std::vector<std::shared_ptr<std::pair<const KeyType,
                         ValueType>>>::iterator table_it) :
                used_it(used_it), used_end_it(used_end_it), table_it(table_it) {
            next();
        }

        std::pair<const KeyType, ValueType>& operator*() {
            return **table_it;
        }

        std::pair<const KeyType, ValueType>* operator->() {
            return table_it->get();
        }

        iterator& operator++() {
            if (used_end_it == used_it) {
                used_it++;
                table_it++;
                return *this;
            }
            used_it++;
            table_it++;
            next();
            return *this;
        }

        iterator operator++(int) {
            iterator res = *this;
            operator++();
            return res;
        }

        bool operator==(const iterator& other) {
            return used_it == other.used_it && table_it == other.table_it;
        }

        bool operator!=(const iterator& other) {
            return !(*this == other);
        }
    };

    class const_iterator {
    private:
        std::vector<bool>::const_iterator used_it, used_end_it;
        typename std::vector<std::shared_ptr<std::pair<const KeyType,
                ValueType>>>::const_iterator table_it;

        void next() {
            while (used_end_it != used_it && !*used_it) {
                used_it++;
                table_it++;
            }
        }

    public:
        const_iterator() {}

        const_iterator(std::vector<bool>::const_iterator used_it,
                       std::vector<bool>::const_iterator used_end_it,
                       typename std::vector<std::shared_ptr<std::pair<const KeyType,
                               ValueType>>>::const_iterator table_it) :
                used_it(used_it), used_end_it(used_end_it), table_it(table_it) {
            next();
        }

        const std::pair<const KeyType, ValueType>& operator*() {
            return **table_it;
        }

        const std::pair<const KeyType, ValueType>* operator->() {
            return table_it->get();
        }

        const_iterator& operator++() {
            if (used_end_it == used_it) {
                used_it++;
                table_it++;
                return *this;
            }
            used_it++;
            table_it++;
            next();
            return *this;
        }

        const_iterator operator++(int) {
            const_iterator res = *this;
            operator++();
            return res;
        }

        bool operator==(const const_iterator& other) {
            return used_it == other.used_it && table_it == other.table_it;
        }

        bool operator!=(const const_iterator& other) {
            return !(*this == other);
        }
    };

    void recalculate_capacity() {
        if (!(filled * 8 >= capacity && filled * 2 <= capacity)) {
            capacity = count * 4 + 1;
            filled = count;

            std::vector<bool> prev_used(capacity);
            used.swap(prev_used);
            std::vector<std::shared_ptr<std::pair<const KeyType, ValueType>>> prev_table(capacity);
            table.swap(prev_table);

            for (int i = 0; i < prev_table.size(); ++i) {
                if (prev_used[i]) {
                    int pos = hash(prev_table[i]->first) % capacity;
                    while (table[pos] && !(table[pos]->first == prev_table[i]->first)) {
                        pos++;
                        if (pos >= capacity) {
                            pos = 0;
                        }
                    }
                    if (!used[pos]) {
                        table[pos] = prev_table[i];
                        used[pos] = true;
                    }
                }
            }
        }
    }

    HashMap(Hash hash = Hash()) : count(0), filled(0), capacity(1), hash(hash) {
        used.resize(1);
        table.resize(1);
    }

    template <typename PairIterator>
    HashMap(PairIterator begin, PairIterator end,
            Hash hash = Hash()) : count(0), filled(0), capacity(1), hash(hash) {
        used.resize(1);
        table.resize(1);
        for (auto it = begin; it != end; ++it) {
            insert(*it);
        }
    }

    HashMap(std::initializer_list<std::pair<KeyType, ValueType>> list,
            Hash hash = Hash()) : count(0), filled(0), capacity(1), hash(hash) {
        used.resize(1);
        table.resize(1);
        for (auto x : list) {
            insert(x);
        }
    }

    size_t size() const {
        return count;
    }

    bool empty() const {
        return count == 0;
    }

    Hash hash_function() const {
        return hash;
    }

    void insert(const std::pair<KeyType, ValueType>& x) {
        int pos = hash(x.first) % capacity;
        while (table[pos] && !(table[pos]->first == x.first)) {
            pos++;
            if (pos >= capacity) {
                pos = 0;
            }
        }
        if (!used[pos]) {
            if (!table[pos]) {
                filled++;
            }
            table[pos] = std::make_shared<std::pair<const KeyType, ValueType>>(x);
            used[pos] = true;
            count++;
        }
        recalculate_capacity();
    }

    void erase(const KeyType& key) {
        int pos = hash(key) % capacity;
        while (table[pos] && !(table[pos]->first == key)) {
            pos++;
            if (pos >= capacity) {
                pos = 0;
            }
        }
        if (used[pos]) {
            used[pos] = false;
            count--;
        }
        recalculate_capacity();
    }

    iterator begin() {
        return iterator(used.begin(), used.end(), table.begin());
    }

    const_iterator begin() const {
        return const_iterator(used.begin(), used.end(), table.begin());
    }

    iterator end() {
        return iterator(used.end(), used.end(), table.end());
    }

    const_iterator end() const {
        return const_iterator(used.end(), used.end(), table.end());
    }

    iterator find(const KeyType& key) {
        int pos = hash(key) % capacity;
        while (table[pos] && !(table[pos]->first == key)) {
            pos++;
            if (pos >= capacity) {
                pos = 0;
            }
        }
        if (used[pos]) {
            return iterator(used.begin() + pos, used.end(), table.begin() + pos);
        } else {
            return end();
        }
    }

    const_iterator find(const KeyType& key) const {
        int pos = hash(key) % capacity;
        while (table[pos] && !(table[pos]->first == key)) {
            pos++;
            if (pos >= capacity) {
                pos = 0;
            }
        }
        if (used[pos]) {
            return const_iterator(used.begin() + pos, used.end(), table.begin() + pos);
        } else {
            return end();
        }
    }

    ValueType& operator[](const KeyType& key) {
        insert({key, ValueType()});
        auto it = find(key);
        return it->second;
    }

    const ValueType& at(const KeyType& key) const {
        auto it = find(key);
        if (it == end()) {
            throw std::out_of_range("error");
        } else {
            return it->second;
        }
    }

    void clear() {
        count = 0;
        filled = 0;
        capacity = 1;
        used.clear();
        used.resize(1);
        table.clear();
        table.resize(1);
    }
};

using namespace std;

int main() {
    ios_base::sync_with_stdio(0);
    cin.tie(0);
    int n;
    cin >> n;
    auto hash = [](int x) {
        long long a = 1e9 + 7;
        long long b = 200720077ll;
        return (b * x) % a;
    };
    HashMap<int, int, decltype(hash)> map(hash);
    while (n) {
        --n;
        char c;
        cin >> c;
        if (c == '+') {
            int key, val;
            cin >> key >> val;
            map[key] = val;
        } else if (c == '-') {
            int key;
            cin >> key;
            map.erase(key);
        } else if (c == '?') {
            int key;
            cin >> key;
            auto it = map.find(key);
            if (it == map.end()) {
                cout << -1 << "\n";
            } else {
                cout << it->second << "\n";
            }
        } else if (c == '<') {
            vector<pair<int, int>> v;
            for (auto x : map) {
                cout << x.first << " " << x.second << "\n";
            }
        } else if (c == '!') {
            map.clear();
        }
    }
    cout << map.size() << "\n";
}
