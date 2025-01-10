# robius.rs

Official website for Project Robius:

[https://robius.rs](https://robius.rs)

## Instructions for building and running locally
1. [Install Zola](https://www.getzola.org/documentation/getting-started/installation/).
2. After you clone this repo, ensure you have all the git submodules:
    ```sh
    git submodule update --init --recursive
    ```
3. Run `zola serve` and then open the printed URL, 
   which is typically [http://127.0.0.1:1111](http://127.0.0.1:1111).
   * When you save changes to the repo, the content will auto-refresh as long as `zola serve` is still running.
