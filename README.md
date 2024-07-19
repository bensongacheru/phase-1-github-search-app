document.addEventListener('DOMContentLoaded', () => {
  const form = document.getElementById('search-form');
  const searchInput = document.getElementById('search-input');
  const searchResults = document.getElementById('search-results');

  form.addEventListener('submit', event => {
    event.preventDefault();
    const searchTerm = searchInput.value.trim();
    if (searchTerm === '') return;

    // Clear previous results
    searchResults.innerHTML = '';

    // Search GitHub users
    searchUsers(searchTerm)
      .then(users => {
        users.forEach(user => {
          const userCard = createUserCard(user);
          searchResults.appendChild(userCard);
        });
      })
      .catch(error => {
        console.error('Error searching users:', error);
      });
  });

  function searchUsers(query) {
    const url = `https://api.github.com/search/users?q=${query}`;
    return fetch(url, {
      headers: {
        Accept: 'application/vnd.github.v3+json'
      }
    })
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        return response.json();
      })
      .then(data => data.items) // Extract the users from the response
      .catch(error => {
        console.error('Error fetching GitHub users:', error);
        return []; // Return empty array on error
      });
  }

  function createUserCard(user) {
    const card = document.createElement('div');
    card.classList.add('user-card');

    const avatar = document.createElement('img');
    avatar.src = user.avatar_url;
    avatar.alt = `${user.login} avatar`;
    avatar.classList.add('avatar');
    card.appendChild(avatar);

    const username = document.createElement('p');
    username.textContent = user.login;
    card.appendChild(username);

    const profileLink = document.createElement('a');
    profileLink.href = user.html_url;
    profileLink.textContent = 'Profile';
    profileLink.target = '_blank';
    card.appendChild(profileLink);

    // Add click event to fetch user's repositories
    card.addEventListener('click', () => {
      fetchUserRepositories(user.login);
    });

    return card;
  }

  function fetchUserRepositories(username) {
    const url = `https://api.github.com/users/${username}/repos`;
    fetch(url, {
      headers: {
        Accept: 'application/vnd.github.v3+json'
      }
    })
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP error! Status: ${response.status}`);
        }
        return response.json();
      })
      .then(repositories => {
        displayUserRepositories(repositories);
      })
      .catch(error => {
        console.error(`Error fetching ${username}'s repositories:`, error);
      });
  }

  function displayUserRepositories(repositories) {
    const repositoriesList = document.createElement('ul');
    repositoriesList.classList.add('repositories-list');
    
    repositories.forEach(repo => {
      const repoItem = document.createElement('li');
      repoItem.textContent = repo.name;
      repositoriesList.appendChild(repoItem);
    });

    // Clear previous repositories
    searchResults.innerHTML = '';
    searchResults.appendChild(repositoriesList);
  }
});
