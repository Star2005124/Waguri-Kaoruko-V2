const { gmd, copyFolderSync } = require("../gift");
const axios = require("axios");
const fs = require("fs");
const path = require("path");
const AdmZip = require("adm-zip");

gmd(
    {
        pattern: "update",
        alias: ["updatenow", "updt", "sync", "update now"],
        react: "🆕",
        desc: "Update the bot to the latest version.",
        category: "owner",
        filename: __filename,
    },
    async (from, Gifted, conText) => {
        const {
            q,
            mek,
            react,
            reply,
            isSuperUser,
            setCommitHash,
            getCommitHash,
            giftedRepo,
        } = conText;

        if (!isSuperUser) {
            await react("❌");
            return reply("❌ Owner Only Command!");
        }

        try {
            await reply("🔍 Checking for New Updates...");

            const { data: commitData } = await axios.get(
                `https://api.github.com/repos/${giftedRepo}/commits/main`,
            );
            const latestCommitHash = commitData.sha;

            const currentHash = await getCommitHash();

            if (latestCommitHash === currentHash) {
                return reply("✅ Your Bot is Already on the Latest Version!");
            }

            const authorName = commitData.commit.author.name;
            const authorEmail = commitData.commit.author.email;
            const commitDate = new Date(
                commitData.commit.author.date,
            ).toLocaleString();
            const commitMessage = commitData.commit.message;

            await reply(
                `🔄 Updating Bot...\n\n*Commit Details:*\n👤 Author: ${authorName} (${authorEmail})\n📅 Date: ${commitDate}\n💬 Message: ${commitMessage}`,
            );

            const zipPath = path.join(__dirname, "..", "atassa-md-main.zip"); // Replace this  with your bot name and branch if you're cloning
            const { data: zipData } = await axios.get(
                `https://github.com/${giftedRepo}/archive/main.zip`,
                { responseType: "arraybuffer" },
            );
            fs.writeFileSync(zipPath, zipData);

            const extractPath = path.join(__dirname, "..", "latest");
            const zip = new AdmZip(zipPath);
            zip.extractAllTo(extractPath, true);

            const sourcePath = path.join(extractPath, "atassa-main"); // Replace this  with your bot name and branch if you're cloning
            const destinationPath = path.join(__dirname, "..");

            const excludeList = [
                ".env",
                "gift/database/database.db",
                "gift/session/session.db",
            ];

            copyFolderSync(sourcePath, destinationPath, excludeList);
            await setCommitHash(latestCommitHash);

            fs.unlinkSync(zipPath);
            fs.rmSync(extractPath, { recursive: true, force: true });

            await reply("✅ Update Complete! Bot is Restarting...");

            setTimeout(() => {
                process.exit(0);
            }, 2000);
        } catch (error) {
            console.error("Update error:", error);
            return reply(
                "❌ Update Failed. Please try by Redeploying Manually.",
            );
        }
    },
);
