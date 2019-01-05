<!-- TOC -->

- [1. Wrapper / Facade](#1-wrapper--facade)
  - [1.1. Code sample](#11-code-sample)
  - [1.2. Notes](#12-notes)
  - [1.3. Refs](#13-refs)

<!-- /TOC -->

# 1. Wrapper / Facade

When you need to provide simplified interface to other complex service or library

## 1.1. Code sample
```
class AppError extends Error {}

class OmdbClient {}

class OmdbParamBuilder {}

class Actor {
  constructor(fullName) {
    this.fullName = fullName;
  }
}

class Movie {
  constructor(title, releaseDate) {
    this.tile = title;
    this.releaseDate = releaseDate;
  }
}

class OmdbApiWraapper {
  constructor(apiKey) {
    this.apiKey = apiKey;
  }

  async getMovieByTitle(title) {
    const pb = new OmdbParamBuilder();
    pb.byTitle(title);
    const params = pb.build();

    const omdb = new OmdbClient(this.apiKey);
    const result = await omdb.get(params);
    if (!result) {
      throw new AppError('movie not found');
    }

    const actors = result.Actors.split(',').map(name => new Actor(name));
    const releaseDate = new Date(Date.parse(result.Released));
    return new Movie(result.Title, releaseDate, actors);
  }
}
```

## 1.2. Notes
- encapsulates library to make easy interaction with this library
- simplifying interface is its main responsibility
- expressive method naming
- prefers static methods over instance methods

## 1.3. Refs
- https://medium.com/selleo/essential-rubyonrails-patterns-clients-and-wrappers-c19320bcda0
