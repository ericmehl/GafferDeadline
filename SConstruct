##########################################################################
#
#  Copyright (c) 2024, Hypothetical Inc. All rights reserved.
#
#  Redistribution and use in source and binary forms, with or without
#  modification, are permitted provided that the following conditions are
#  met:
#
#      * Redistributions of source code must retain the above
#        copyright notice, this list of conditions and the following
#        disclaimer.
#
#      * Redistributions in binary form must reproduce the above
#        copyright notice, this list of conditions and the following
#        disclaimer in the documentation and/or other materials provided with
#        the distribution.
#
#      * Neither the name of Hypothetical Inc. nor the names of
#        any other contributors to this software may be used to endorse or
#        promote products derived from this software without specific prior
#        written permission.
#
#  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
#  IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
#  THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
#  PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR
#  CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
#  EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
#  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
#  PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
#  LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
#  NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
#  SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
##########################################################################

import distutils.dir_util
import os
import pathlib
import shutil
import sys

EnsureSConsVersion(3, 0, 2)  # Substfile is a default builder as of 3.0.2

###############################################################################################
# Version
###############################################################################################

# for announcing major milestones - may contain all of the below
gafferDeadlineMilestoneVersion = 0
gafferDeadlineMajorVersion = 57  # backwards-incompatible changes
gafferDeadlineMinorVersion = 3  # new backwards-compatible features
gafferDeadlinePatchVersion = 0  # bug fixes
gafferDeadlineVersionSuffix = ""  # beta, alpha, etc.

###############################################################################################
# Command line options
###############################################################################################

optionsFile = None
if "GAFFER_OPTIONS_FILE" in os.environ:
    optionsFile = os.environ["GAFFERDEADLINE_OPTIONS_FILE"]

if "OPTIONS" in ARGUMENTS:
    optionsFile = ARGUMENTS["OPTIONS"]

options = Variables(optionsFile, ARGUMENTS)

options.Add(
    "BUILD_DIR",
    "The destination directory in which the build will be made.",
    str(
        pathlib.Path("build")
        / "gafferDeadline-${GAFFERDEADLINE_MILESTONE_VERSION}.${GAFFERDEADLINE_MAJOR_VERSION}.${GAFFERDEADLINE_MINOR_VERSION}.${GAFFERDEADLINE_PATCH_VERSION}${GAFFERDEADLINE_VERSION_SUFFIX}"
    ),
)

options.Add(
    "INSTALL_DIR",
    "The destination directory for the installation.",
    str(
        pathlib.Path("install")
        / "gafferDeadline-${GAFFERDEADLINE_MILESTONE_VERSION}.${GAFFERDEADLINE_MAJOR_VERSION}.${GAFFERDEADLINE_MINOR_VERSION}.${GAFFERDEADLINE_PATCH_VERSION}${GAFFERDEADLINE_VERSION_SUFFIX}"
    ),
)

options.Add(
    "DEADLINE_PLUGIN_REPOSITORY",
    'The directory for the Deadline repository. The "install" build type will copy the'
    "Deadline Gaffer plugin to this location.",
    str(
        pathlib.Path("install")
        / "gafferDeadline-${GAFFERDEADLINE_MILESTONE_VERSION}.${GAFFERDEADLINE_MAJOR_VERSION}.${GAFFERDEADLINE_MINOR_VERSION}.${GAFFERDEADLINE_PATCH_VERSION}${GAFFERDEADLINE_VERSION_SUFFIX}"
        / "custom"
    ),
)

options.Add(
    "DEPENDENCY_SCRIPT_DIR",
    "The directory for the `gaffer_batch_dependency.py` script. For production, this must"
    "be accessible to Deadline Workers.",
    str(
        pathlib.Path("install")
        / "gafferDeadline-${GAFFERDEADLINE_MILESTONE_VERSION}.${GAFFERDEADLINE_MAJOR_VERSION}.${GAFFERDEADLINE_MINOR_VERSION}.${GAFFERDEADLINE_PATCH_VERSION}${GAFFERDEADLINE_VERSION_SUFFIX}"
    ),
)

options.Add(
    "PACKAGE_FILE",
    "The file in which the final GafferDeadline file will be created by the package script.",
    "${INSTALL_DIR}.tar.gz" if sys.platform != "win32" else "${INSTALL_DIR}.zip",
)

options.Add(
    "GAFFERDEADLINE_MILESTONE_VERSION",
    "Milestone version",
    str(gafferDeadlineMilestoneVersion),
)
options.Add(
    "GAFFERDEADLINE_MAJOR_VERSION", "Major version", str(gafferDeadlineMajorVersion)
)
options.Add(
    "GAFFERDEADLINE_MINOR_VERSION", "Minor version", str(gafferDeadlineMinorVersion)
)
options.Add(
    "GAFFERDEADLINE_PATCH_VERSION", "Patch version", str(gafferDeadlinePatchVersion)
)
options.Add(
    "GAFFERDEADLINE_VERSION_SUFFIX", "Version suffix", str(gafferDeadlineVersionSuffix)
)

env = Environment(options=options)

libraries = {
    "GafferDeadline": {"apps": ["dispatch", "execute"]},
    "GafferDeadlineTest": {
        "additionalFiles": (pathlib.Path("python") / "GafferDeadlineTest").glob("**/*"),
        "apps": ["dispatch", "execute"],
    },
    "GafferDeadlineUI": {"apps": ["gui"]},
    "GafferDeadlineUITest": {
        "additionalFiles": (pathlib.Path("python") / "GafferDeadlineUITest").glob(
            "**/*"
        ),
        "apps": ["gui"],
    },
}

for libraryName, libraryDef in libraries.items():
    libEnv = env.Clone()

    buildPath = pathlib.Path(libraryDef.get("installRoot", "$BUILD_DIR"))

    fileSubstitutions = {
        "!GAFFERDEADLINE_MILESTONE_VERSION!": libEnv.subst(
            "$GAFFERDEADLINE_MILESTONE_VERSION"
        ),
        "!GAFFERDEADLINE_MAJOR_VERSION!": libEnv.subst(
            "$GAFFERDEADLINE_MILESTONE_VERSION"
        ),
        "!GAFFERDEADLINE_MINOR_VERSION!": libEnv.subst(
            "$GAFFERDEADLINE_MILESTONE_VERSION"
        ),
        "!GAFFERDEADLINE_PATCH_VERSION!": libEnv.subst(
            "$GAFFERDEADLINE_MILESTONE_VERSION"
        ),
        "!GAFFERDEADLINE_VERSION_SUFFIX!": libEnv.subst(
            "$GAFFERDEADLINE_MILESTONE_VERSION"
        ),
    }

    pythonFiles = list((pathlib.Path("python") / libraryName).glob("**/*.py*"))

    for pythonFile in pythonFiles:
        pythonFileInstall = env.Substfile(
            str(buildPath / (str(pythonFile).replace(".in", ""))),
            str(pythonFile),
            SUBST_DICT=fileSubstitutions,
        )
        env.Alias("build", pythonFileInstall)

    for startupDir in libraryDef.get("apps", []) + [libraryName]:
        for startupFile in (pathlib.Path("startup") / startupDir).glob("*.py"):
            startupFileInstall = env.InstallAs(
                str(buildPath / startupFile), str(startupFile)
            )
            env.Alias("build", startupFileInstall)

    for additionalFile in libraryDef.get("additionalFiles", []):
        print(additionalFile, list(pythonFiles))
        if additionalFile in pythonFiles:
            continue
        additionalFileInstall = env.InstallAs(
            str(buildPath / additionalFile), str(additionalFile)
        )
        env.Alias("build", additionalFileInstall)

deadlineGafferPluginInstall = env.InstallAs(
    "$DEADLINE_PLUGIN_REPOSITORY/Gaffer", "custom/Gaffer"
)
env.Alias("build", deadlineGafferPluginInstall)

dependencyScriptInstall = env.InstallAs(
    "$DEPENDENCY_SCRIPT_DIR/gaffer_batch_dependency.py",
    "gaffer_batch_dependency.py"
)

#########################################################################################################
# Installation
#########################################################################################################


def installer(target, source, env):
    distutils.dir_util.copy_tree(
        str(source[0]), str(target[0]), preserve_symlinks=True, update=True
    )


install = env.Command("$INSTALL_DIR", "$BUILD_DIR", installer)
env.AlwaysBuild(install)
env.NoCache(install)

env.Alias("install", install)

#########################################################################################################
# Packaging
#########################################################################################################


def packager(target, source, env):
    target = str(target[0])
    source = str(source[0])
    b = os.path.basename(source)
    d = os.path.dirname(source)

    shutil.make_archive(
        os.path.splitext(target)[0],
        "zip" if target.endswith(".zip") else "gztar",
        root_dir=d,
        base_dir=b,
    )


package = env.Command("$PACKAGE_FILE", "$INSTALL_DIR", packager)
env.NoCache(package)
env.Alias("package", package)
