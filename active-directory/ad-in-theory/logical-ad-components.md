# 💫 Logical AD Components

In the enchanted realm of Active Directory, the magic unfolds through logical components that define its structure and relationships. Let's embark on a journey to understand these mystical elements.

### 1. **Class Objects 🌐**

#### - **Definition:**

Class objects in Active Directory define the blueprint for various types of directory objects, such as users, groups, and computers. They specify the attributes an object can have and the rules that govern them.

#### - **Analogy:**

Imagine class objects as magical scrolls detailing the characteristics and magical properties that different entities in your kingdom can possess.

<figure><img src="https://imgs.search.brave.com/zIVj_yv0Efme778zJOsnAnXPFZoLYiRyfVzsks5GQTc/rs:fit:860:0:0/g:ce/aHR0cHM6Ly93d3cu/dGVjaC1mYXEuY29t/L3dwLWNvbnRlbnQv/dXBsb2Fkcy9hY3Rp/dmUtZGlyZWN0b3J5/LW9iamVjdHMuanBn" alt=""><figcaption></figcaption></figure>

### 2. **Attribute Objects 📜**

#### - **Definition:**

Attribute objects define the specific properties or characteristics of an Active Directory object. For example, a user object may have attributes like "name," "email," or "title."

#### - **Visualization:**

Think of attribute objects as the magical runes inscribed on each entity, describing their unique qualities and attributes.

### 3. **Domains 🌐**

#### - **Definition:**

A domain in Active Directory is a logical grouping of objects, such as users and computers, that share a common namespace. It acts as a security boundary, and each domain has its own unique name.

#### - **Analogy:**

Envision domains as distinct realms in your magical universe, each with its own set of inhabitants and rules.

### 4. **Trees 🌲**

#### - **Definition:**

A tree in Active Directory is a collection of one or more domains that share a contiguous namespace. Domains within a tree trust each other and form a hierarchical structure.

#### - **Visualization:**

Picture a magnificent tree with branches representing individual domains, all interconnected and sharing the same magical energy.

<figure><img src="https://imgs.search.brave.com/md2To4-CiTKIgB9gFWHMtMmvjqTrrIFx-3HsJg8jDSA/rs:fit:860:0:0/g:ce/aHR0cHM6Ly9pMi53/cC5jb20vaXB3aXRo/ZWFzZS5jb20vd3At/Y29udGVudC91cGxv/YWRzLzIwMjAvMDEv/V2hhdC1pcy1hLVRS/RUUtaW4tQWN0aXZl/LURpcmVjdG9yeS12/MC4xYi5qcGc_Zml0/PTgwMCw0OTgmc3Ns/PTE" alt=""><figcaption></figcaption></figure>

### 5. **Forest 🌳**

#### - **Definition:**

A forest is a collection of one or more trees that do not form a contiguous namespace. Trust relationships exist between trees in a forest, allowing for communication and collaboration.

#### - **Analogy:**

Imagine a vast magical forest where each tree represents a domain, and the entire ecosystem thrives on harmony and cooperation.

<figure><img src="https://imgs.search.brave.com/EuHcZEv7k_fLVUMvMbf1o2UKvLasJvvnntwcUubAI-E/rs:fit:860:0:0/g:ce/aHR0cHM6Ly9pbmZv/LnZhcm9uaXMuY29t/L2hzLWZzL2h1YmZz/L0ltcG9ydGVkX0Js/b2dfTWVkaWEvZG9t/YWluLWZvcmVzdEAy/eC0xLnBuZz93aWR0/aD0zMDAwJmhlaWdo/dD0yNjY0Jm5hbWU9/ZG9tYWluLWZvcmVz/dEAyeC0xLnBuZw" alt=""><figcaption></figcaption></figure>

### 6. **Organization Units (OUs) 🏰**

#### - **Definition:**

OUs are containers within domains used to organize and manage objects. They provide a way to delegate administrative control and apply Group Policies.

#### - **Visualization:**

Think of OUs as magical strongholds within your kingdom, each with its own set of rules and overseers.

<figure><img src="https://imgs.search.brave.com/WdYwwihJXwbAz9ewtMqvLlVgkUITFYHnEg50OcSRUWM/rs:fit:860:0:0/g:ce/aHR0cHM6Ly93d3cu/dGVjaC1mYXEuY29t/L3dwLWNvbnRlbnQv/dXBsb2Fkcy9kZXBs/b3lpbmctc29mdHdh/cmUtdGhyb3VnaC1n/cm91cC1wb2xpY3ku/anBn" alt=""><figcaption><p>Making rules for all Users and all Computers in different sections</p></figcaption></figure>

### 7. **Trust (Directional, Transitive) 🤝**

#### - **Definition:**

Trust in Active Directory represents the relationships between domains or forests. It allows entities in one domain to access resources in another.

<figure><img src="https://imgs.search.brave.com/e4Jkf8EkPzQkGOKfzlVVKlf-x1LmS8EdbPFvlA2jKLI/rs:fit:860:0:0/g:ce/aHR0cHM6Ly9sZWFy/bi5taWNyb3NvZnQu/Y29tL2VuLXVzL2Vu/dHJhL2lkZW50aXR5/L2RvbWFpbi1zZXJ2/aWNlcy9tZWRpYS9j/b25jZXB0cy1mb3Jl/c3QtdHJ1c3QvdHJ1/c3QtcmVsYXRpb25z/aGlwcy5wbmc" alt=""><figcaption></figcaption></figure>

#### - **Analogy:**

Picture trust as magical gateways or portals connecting different realms, enabling seamless interaction and cooperation.

<figure><img src="https://imgs.search.brave.com/JSoiJtpjHyUpOfxZr6I2RIhWUWGErEtLNZKuNXjyL2A/rs:fit:860:0:0/g:ce/aHR0cHM6Ly9hY2Nl/c3MucmVkaGF0LmNv/bS93ZWJhc3NldHMv/YXZhbG9uL2QvUmVk/X0hhdF9FbnRlcnBy/aXNlX0xpbnV4LTct/V2luZG93c19JbnRl/Z3JhdGlvbl9HdWlk/ZS1lbi1VUy9pbWFn/ZXMvZGQzZTk0ZWNi/MGE2YzU2YmI3ZDNm/NjdjOWU0MDMxMGYv/dHJ1c3QtZGlyZWN0/aW9uLnBuZw" alt=""><figcaption></figcaption></figure>
