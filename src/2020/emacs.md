# Emacs 配置 #
<2020-7-9> _LanderX_ [返回首页](https://lander-hatsune.github.io/)

> GNU Emacs: An extensible, customizable, free/libre text editor — and more.

### 当前配置文件（2020年7月9日） ###

#### `~/.emacs` ####

``` emacs-lisp
;; packages (melpa stable)
(require 'package)
(add-to-list 'package-archives
             '("melpa-stable" . "https://stable.melpa.org/packages/") t)
(package-initialize)


;; basic face
(tool-bar-mode 0)
(menu-bar-mode 0)
(scroll-bar-mode 0)
(global-linum-mode 1)

;;; English Font
(set-face-attribute
'default nil :font "Ubuntu Mono 15")

;;; Chinese Font
(dolist (charset '(kana han symbol cjk-misc bopomofo))
(set-fontset-font (frame-parameter nil 'font)
charset
(font-spec :family "SimSun" :size 22)))

;;; zenburn theme
(load-theme 'zenburn t)

;; sanitic-tomorrow theme
;;(require 'color-theme-sanityinc-tomorrow)

;; space indent
(setq default-tab-width 4)
(setq-default indent-tabs-mode nil)
(setq c-default-style "Linux")
(setq c-basic-offset 4)

;; shut welcome face
(setq inhibit-splash-screen 1)


;; flycheck
(use-package flycheck
  :ensure t
  :init (global-flycheck-mode))


;; orgmode
(require 'org)
(add-to-list 'auto-mode-alist '("\\.org$" . org-mode))
(global-set-key "\C-cl" 'org-store-link)
(global-set-key "\C-ca" 'org-agenda)
(global-font-lock-mode 1)
(require 'org-tempo)
(setq org-log-done t)
(setq org-src-fontify-natively t)

;;; htmlize
;;; use minted to highlight code in latex
(require 'ox-latex)
(add-to-list 'org-latex-packages-alist '("" "minted"))
(setq org-latex-listings 'minted)


;; markdown mode
(use-package markdown-mode
  :ensure t
  :commands (markdown-mode gfm-mode)
  :mode (("README\\.md\\'" . gfm-mode)
         ("\\.md\\'" . markdown-mode)
         ("\\.markdown\\'" . markdown-mode))
  :init (setq markdown-command "multimarkdown"))


(provide '.emacs)
;;; .emacs ends here
```

#### `~/.emacs.d/init.el` ####

``` emacs-lisp
;;; package --- summary:
;;; code:
;;; commentary:
;; above for flycheck
(load "~/.emacs")

(provide '.emacs)
;;; init.el ends here
```

### 其他 & 参考资源 ###

- [auto-complete](https://github.com/auto-complete/auto-complete)
- [org-mode](https://orgmode.org/)
- [markdown-mode](https://jblevins.org/projects/markdown-mode/)

