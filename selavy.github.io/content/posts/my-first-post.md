+++
title = "My First Post"
date = 2019-01-19T19:35:59-08:00
draft = false
tags = ["tests"]
categories = []
+++
This is a test first post. A change!

This is an example using code highlighting for HTML:
{{< highlight html >}}
<section id="main">
  <div>
    <h1 id="title">{{ .Title }}</h1>
    {{ range .Pages }}
      {{ .Render "summary"}}
    {{ end }}
  </div>
</section>
{{< /highlight >}}

Here is an example using code highlighting for C++:
{{< highlight cpp >}}
template <class Key, class Value>
class HashTable {
public:
    using value_type = Value;
    using key_type = const Key;

    constexpr HashTable();

    ~HashTable() noexcept;

    Value& operator[](const Key) const noexcept;

    [[nodiscard]]
    constexpr size_type size() const noexcept {
        return _size;
    }

private:
    size_type _size;
    T* _values;
};
{{< /highlight >}}