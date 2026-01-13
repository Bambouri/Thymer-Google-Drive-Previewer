class Plugin extends AppPlugin {

    onLoad() {
        this.startObserver();
        console.log("Drive Previewer loaded!");
    }

    startObserver() {
        const self = this;
        
        this.observer = new MutationObserver(function() {
            self.processNode(document.body);
        });

        this.observer.observe(document.body, {
            childList: true,
            subtree: true,
            attributes: true,
            characterData: true
        });

        setTimeout(function() {
            self.processNode(document.body);
        }, 500);

        setInterval(function() {
            self.processNode(document.body);
        }, 1000);
    }

    processNode(root) {
        const self = this;
        if (!root.querySelectorAll) return;
        
        const links = root.querySelectorAll("a:not([data-drive-processed])");
        
        links.forEach(function(link) {
            const href = link.getAttribute("href") || "";
            const text = link.textContent || "";
            
            const driveInfo = self.parseGoogleDriveLink(href) || self.parseGoogleDriveLink(text);
            
            if (driveInfo) {
                self.styleAsDriveLink(link, driveInfo);
            }
        });
    }

    parseGoogleDriveLink(url) {
        if (!url) return null;
        
        // Google Drive file: drive.google.com/file/d/FILE_ID/view
        const fileMatch = url.match(/drive\.google\.com\/file\/d\/([a-zA-Z0-9_-]+)/);
        if (fileMatch) {
            return {
                type: "file",
                id: fileMatch[1],
                url: url,
                embedUrl: "https://drive.google.com/file/d/" + fileMatch[1] + "/preview"
            };
        }
        
        // Google Docs: docs.google.com/document/d/DOC_ID/edit
        const docMatch = url.match(/docs\.google\.com\/document\/d\/([a-zA-Z0-9_-]+)/);
        if (docMatch) {
            return {
                type: "doc",
                id: docMatch[1],
                url: url,
                embedUrl: "https://docs.google.com/document/d/" + docMatch[1] + "/preview"
            };
        }
        
        // Google Sheets: docs.google.com/spreadsheets/d/SHEET_ID/edit
        const sheetMatch = url.match(/docs\.google\.com\/spreadsheets\/d\/([a-zA-Z0-9_-]+)/);
        if (sheetMatch) {
            return {
                type: "sheet",
                id: sheetMatch[1],
                url: url,
                embedUrl: "https://docs.google.com/spreadsheets/d/" + sheetMatch[1] + "/preview"
            };
        }
        
        // Google Slides: docs.google.com/presentation/d/SLIDE_ID/edit
        const slideMatch = url.match(/docs\.google\.com\/presentation\/d\/([a-zA-Z0-9_-]+)/);
        if (slideMatch) {
            return {
                type: "slide",
                id: slideMatch[1],
                url: url,
                embedUrl: "https://docs.google.com/presentation/d/" + slideMatch[1] + "/embed"
            };
        }
        
        return null;
    }

    getFileType(driveInfo, label) {
        // For docs/sheets/slides, we know the type
        if (driveInfo.type === "doc") return "document";
        if (driveInfo.type === "sheet") return "document";
        if (driveInfo.type === "slide") return "document";
        
        // For drive files, try to guess from label
        const lowerLabel = label.toLowerCase();
        
        if (lowerLabel.includes("audio") || lowerLabel.includes("mp3") || lowerLabel.includes("wav") || lowerLabel.includes("m4a")) {
            return "audio";
        }
        if (lowerLabel.includes("video") || lowerLabel.includes("mp4") || lowerLabel.includes("mov") || lowerLabel.includes("avi")) {
            return "video";
        }
        if (lowerLabel.includes("pdf") || lowerLabel.includes("doc") || lowerLabel.includes("sheet") || lowerLabel.includes("slide")) {
            return "document";
        }
        
        // Default to document for better viewing
        return "document";
    }

    getTypeIcon(fileType) {
        switch (fileType) {
            case "audio":
                return '<svg width="14" height="14" viewBox="0 0 24 24" fill="#f5f0e6"><path d="M12 3v10.55c-.59-.34-1.27-.55-2-.55-2.21 0-4 1.79-4 4s1.79 4 4 4 4-1.79 4-4V7h4V3h-6z"/></svg>';
            case "video":
                return '<svg width="14" height="14" viewBox="0 0 24 24" fill="#f5f0e6"><path d="M8 5v14l11-7z"/></svg>';
            case "document":
            default:
                return '<svg width="14" height="14" viewBox="0 0 24 24" fill="#f5f0e6"><path d="M14 2H6c-1.1 0-2 .9-2 2v16c0 1.1.9 2 2 2h12c1.1 0 2-.9 2-2V8l-6-6zm2 16H8v-2h8v2zm0-4H8v-2h8v2zm-3-5V3.5L18.5 9H13z"/></svg>';
        }
    }

    styleAsDriveLink(linkElement, driveInfo) {
        linkElement.setAttribute("data-drive-processed", "true");
        linkElement.dataset.driveType = driveInfo.type;
        linkElement.dataset.driveId = driveInfo.id;
        linkElement.dataset.driveUrl = driveInfo.url;
        linkElement.dataset.embedUrl = driveInfo.embedUrl;
        
        linkElement.classList.add("thymer-drive-link");
        
        const parent = linkElement.parentElement;
        let label = "";
        if (parent) {
            const fullText = parent.textContent || "";
            const urlStart = fullText.indexOf("https://");
            if (urlStart > 0) {
                label = fullText.substring(0, urlStart).replace(/[:\-\s]+$/, "").trim();
            }
        }
        if (!label) {
            // Default labels based on type
            switch (driveInfo.type) {
                case "doc": label = "Google Doc"; break;
                case "sheet": label = "Google Sheet"; break;
                case "slide": label = "Google Slides"; break;
                default: label = "Drive File";
            }
        }
        linkElement.dataset.label = label;

        const self = this;

        linkElement.addEventListener("click", function(e) {
            e.preventDefault();
            e.stopPropagation();
            self.showPreview(driveInfo, label);
        });

        linkElement.addEventListener("contextmenu", function(e) {
            e.preventDefault();
            e.stopPropagation();
            self.showContextMenu(e, driveInfo, label);
        });
    }

    showContextMenu(e, driveInfo, label) {
        const existing = document.querySelector(".thymer-drive-context-menu");
        if (existing) existing.remove();

        const menu = document.createElement("div");
        menu.className = "thymer-drive-context-menu";
        menu.style.left = e.clientX + "px";
        menu.style.top = e.clientY + "px";

        const self = this;

        menu.innerHTML = '<button data-action="preview">Open Preview</button><button data-action="copy">Copy Link</button><button data-action="open">Open in Google Drive</button>';

        menu.querySelector('[data-action="preview"]').addEventListener("click", function() {
            menu.remove();
            self.showPreview(driveInfo, label);
        });

        menu.querySelector('[data-action="copy"]').addEventListener("click", function() {
            navigator.clipboard.writeText(driveInfo.url).then(function() {
                self.ui.addToaster({ title: "Copied!", message: "Link copied to clipboard", dismissible: true, autoDestroyTime: 2000 });
            });
            menu.remove();
        });

        menu.querySelector('[data-action="open"]').addEventListener("click", function() {
            window.open(driveInfo.url, "_blank");
            menu.remove();
        });

        document.body.appendChild(menu);

        setTimeout(function() {
            document.addEventListener("click", function handler() {
                menu.remove();
                document.removeEventListener("click", handler);
            });
        }, 10);
    }

    showPreview(driveInfo, title) {
        const existing = document.getElementById("thymer-drive-modal");
        if (existing) existing.remove();

        const fileType = this.getFileType(driveInfo, title);
        const icon = this.getTypeIcon(fileType);
        
        let modalClass = "thymer-drive-modal";
        let iframeHeight = "80";
        
        switch (fileType) {
            case "audio":
                modalClass += " thymer-drive-modal-audio";
                iframeHeight = "80";
                break;
            case "video":
                modalClass += " thymer-drive-modal-video";
                iframeHeight = "450";
                break;
            case "document":
            default:
                modalClass += " thymer-drive-modal-document";
                iframeHeight = "100%";
                break;
        }

        const modal = document.createElement("div");
        modal.id = "thymer-drive-modal";
        
        modal.innerHTML = '<div class="thymer-drive-modal-backdrop"><div class="' + modalClass + '"><div class="thymer-drive-modal-header"><div class="thymer-drive-title-row"><span class="thymer-drive-title-icon">' + icon + '</span><h3>' + this.escapeHtml(title) + '</h3></div><button class="thymer-drive-modal-close">&times;</button></div><div class="thymer-drive-iframe-wrapper"><iframe src="' + driveInfo.embedUrl + '" width="100%" height="' + iframeHeight + '" allow="autoplay"></iframe></div><p class="thymer-drive-hint">Press Escape or click outside to close</p></div></div>';

        document.body.appendChild(modal);

        modal.querySelector(".thymer-drive-modal-backdrop").addEventListener("click", function(e) {
            if (e.target === this) modal.remove();
        });

        modal.querySelector(".thymer-drive-modal-close").addEventListener("click", function() {
            modal.remove();
        });

        document.addEventListener("keydown", function handler(e) {
            if (e.key === "Escape") {
                modal.remove();
                document.removeEventListener("keydown", handler);
            }
        });
    }

    escapeHtml(text) {
        const div = document.createElement("div");
        div.textContent = text;
        return div.innerHTML;
    }

}
