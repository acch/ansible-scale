Contributing to Ansible-Scale
=============================

Please use the [issue tracker](https://github.com/acch/ansible-scale/issues) to ask questions, report bugs and request features.

Contributing new content and updates
------------------------------------

- Fork the [code](https://github.com/acch/ansible-scale) to your own Git repository
- Make changes in your forked repository
- When you are happy with your updates, submit a [pull request](https://github.com/acch/ansible-scale/pulls) describing the changes
- IMPORTANT: Make sure that your forked repository is in sync with the base repository before sending a pull request
- The updates will be reviewed and merged in

Code guide
----------

- Indent by two spaces

- Always use pure YAML &mdash; i.e. write this:

  ```
  copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
  ```

  ...not this:

  ```
  copy: src=/srv/myfiles/foo.conf dest=/etc/foo.conf
  ```

- Only quote if necessary:

  String starts with a YAML control character: `-`, `{`, `}`, `[`, `]`, `*`, `&`, `?`, `|`, `>`, `!`, `%`, `` `, `#`, `@`, `:`

  String contains colon `:` followed by space

  String is a boolean value which should be preserved as string

- Adhere to standard sequence of declarations:

  ```
  - name: tag | Task description
    vars:
      var-1: ...
      var-2: ...
    task:
      task-param-1: ...
      task-param-2: ...
    notify: ...
    register: ...
    when: ...
    with_items: ...
    run_once: ...
    delegate_to: ...
    delegate_facts: ...
    changed_when: ...
    failed_when: ...
  ```

Copyright and license
---------------------

By contributing, you agree to license your contribution under the [MIT License](LICENSE).
